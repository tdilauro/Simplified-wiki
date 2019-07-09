# Basic concepts

The circulation manager uses a Postgres relational database to store information about real-world concepts: books, authors, genres, libraries, licenses, patrons, loans, holds, and so on. This database is good enough for most purposes, but there are two things it's really bad at:

1. Taking a search query someone typed in (e.g. `italian romance` or `raina telemger`) and reliably finding the books that person is probably looking for.
2. Quickly finding exactly which books belong in a given lane (e.g. finding all the YA science fiction books on the 'staff picks' list).

Unfortunately these are the two most important features of the circulation manager. Nearly every OPDS feed we serve is either a list of books taken from a lane, or a list of search results. We brought in an Elasticsearch server to help us implement those two features.

Our Elasticsearch index stores a reformatted copy of some information from the database. We don't store the whole database. We only store what we need to do these two jobs, and we store it in a way that's optimized for these jobs.

In the Elasticsearch index we primarily store information about works, but we also includes some information about authors, genres, custom lists, and licenses. This is because we might need to find all books by a certain author, or filter out books that have an active holds queue. The circulation manager has a lot of weird ways of generating OPDS feeds, and most of those techniques need special support in the Elasticsearch index.

The Elasticsearch index is disposable. On most sites, it can be deleted and recreated from scratch in under an hour, using information from the Postgres database. It gets rebuilt once a day, in the middle of the night, and periodically it gets deleted and recreated from scratch as a side effect of an upgrade. If you were to destroy the Elasticsearch index, search performance would be degraded for a few minutes and then everything would be fine. 

By contrast, the Postgres database is _not_ disposable. The information necessary to build the search index comes from the Postgres database. So do whatever you want with the Elasticsearch index, but don't mess with the database.

# Troubleshooting a search index

If you manage a circulation manager, it's useful to know how that software makes use of Elasticsearch. Many patron-visible problems can be traced to an out-of-date or misconfigured search index, and you have tools at your disposal to diagnose and fix the problem.

## How the search index is kept updated

When the circulation manager realizes that a work's entry in the Elasticsearch index is out of date, it doesn't reindex the work immediately -- this would slow things down for a librarian who is using the admin interface or a library patron who is trying to borrow a book. Instead, the circulation manager creates a 'registered' entry in the `workcoveragerecords` table for that work. This represents work to be done. It's how the circulation manager remembers that this work's entry in the search index is stale and needs to be refreshed.

The `bin/search_index_refresh` runs every few minutes. It looks for stale works: the ones in the 'registered' state, or the ones with no `workcoveragerecords` entry at all. It reindexes the stale works and changes their `workcoveragerecords` entries to the 'success' state. If there's a problem with a record, it puts that record in the 'transient failure' or 'persistent failure' state.

After handling all the works in the 'registered' state, `bin/search_index_refresh` then tries all the works in the 'transient failure' state, on the assumption that a transient problem might go away on its own.

The `bin/search_index_clear` script runs once a day, in the middle of the night. It removes all search-related records from the `workcoveragerecords`. The next time `bin/search_index_refresh` runs, it will appear as though no search index work has ever been done. At that point the script will update every single work in the system.

This guarantees that the search index is completely rebuilt once a day. Even if there are bugs in the system, no book should ever have a search index entry more than 24 hours out of date.

## Looking at the update queue

You can use the following SQL command to see the size of the search index update queue:

`select status, count(id) from workcoveragerecords where operation='update-search-index' group by status order by status;`

You should see something like this:

```
      status       | count  
-------------------+--------
 success           | 309621
 transient failure |     19
 registered        |    181
```

The number of works in the `success` state should be about the same as the total number of works on your system. If you see a large number of works in the `registered` state, then `bin/search_index_refresh` probably hasn't been running. If you see a large number of works in the `transient failure` or `persistent failure` states, then something is probably wrong with your ElasticSearch server.

If you see nothing, then the most likely explanation is that `bin/search_index_clear` is running but `bin/search_index_refresh` is not.

## Looking in the index

Works are indexed using their database ID. You can always see what your Elasticsearch index thinks of a specific work by making a GET request to this URL:

```
https://{elasticsearch-hostname}/{search-prefix}-v4/work-type/{work-ID}
```

The default search prefix is `circulation-works` (you can change this in the admin interface), so something like this should usually work:

```
https://{elasticsearch-hostname}/circulation-works-v4/work-type/1
```

You should get back a big JSON document full of accurate information about the book. If you get a small document that says `"found":false`, then either the work ID doesn't exist in the database, the search prefix is wrong, or this work was never indexed.

## Completely rebuilding the index

You can completely recreate the search index by running the `bin/repair/search_index` script. This is not run normally, but running it manually is an easy way to see whether your current problem is caused by an out-of-date search index.

# Why not keep everything in Elasticsearch?

Before getting started with the more technical aspects of how we use Elasticsearch, here's a question to consider. Elasticsearch and a relational database are both very complicated pieces of technology. Why do we need both? We already know why we can't do everything in the relational database -- it's really bad at two important jobs. Why can't we do everything in Elasticsearch? There are two main reasons:

First, it's very difficult to write Elasticsearch query code. One of the main purposes of this document is showing you how to write and understand this code! The language for Elasticsearch queries is nowhere near as mature and stable as SQL, the language for relational database queries. The Elasticsearch API changes frequently, and the [Elasticsearch DSL](https://elasticsearch-dsl.readthedocs.io/en/latest/) Python library is just a thin layer around the raw API. 

More importantly, Elasticsearch documents store redundant information, and relational databases don't. This would cause major problems if we were to use Elasticsearch as our main data store instead of a convenient way to run two special types of queries.

If two books have the same author, a relational database stores five pieces of information:

* Book 1 ("Cujo")
* Book 2 ("Misery")
* Author A ("Stephen King")
* Book 1 is written by author A
* Book 2 is written by author A

Our Elasticsearch index mushes all of this together and stores two pieces of information:

* "Cujo" is written by Stephen King.
* "Misery" is written by Stephen King.

This makes it difficult to keep the Elasticsearch index up to date. A small change to the data may mean that a lot of index entries need to be updated. This can be time consuming, and there's also the risk that one of the updates will be missed and one book will have out-of-date information.

So long as the Elasticsearch index is treated as disposable and rebuilt regularly, any missed update problems are minor. If we treated the Elasticsearch index as the canonical location for this information, and two copies of the same information got out of sync, we'd have a big problem: there'd be no way of knowing which copy was correct!

"Keep everything in Elasticsearch" can be the right choice when the application has only one data model object, but the circulation manager has many different objects used for different purposes. The relationships between those objects are best described with a relational database.

# A sample search document

Here's a search document that shows off everything we put into the Elasticsearch index. I'll be using this as a reference throughout the rest of this document. Each of these fields corresponds to something in the database.

```
{
  "_id": 122940, 
  "work_id": 122940,
  "presentation_ready": true,
  "last_update_time": 1561581146,

  "title": "Law of the Mountain Man", 
  "sort_title": "Law of the Mountain Man", 
  "subtitle": "Mountain Man Series, Book 5", 

  "series": "Mountain Man", 
  "series_position": 5, 

  "author": "William W. Johnstone", 
  "sort_author": "Johnstone, William W.", 
  "contributors": [
    {
      "display_name": "William W. Johnstone", 
      "role": "Author", 
      "sort_name": "Johnstone, William W.", 
    }
  ], 

  "medium": "Book", 
  "publisher": "Kensington", 
  "imprint": "Pinnacle", 
  "summary": "<p><B>The Greatest Western Writer Of The 21st Century<P>When The Bullets Start To Fly</B><P>Smoke Jensen sat in a cave and boiled the last of his coffee. He figured he was in Idaho-somewhere south of Montpelier-but he was certain about only two things: he was cold and he was being hunted by a small army of men....", 
  "quality": 0.743,

  "language": "eng", 
  "audience": "Adult", 
  "target_age": {
    "lower": 18, 
    "upper": null
  },
  "fiction": "Fiction", 
  "classifications": [
    {
      "scheme": "http://id.worldcat.org/fast/", 
      "term": "Idaho", 
      "weight": 0.0133
    }, 
    {
      "scheme": "http://id.worldcat.org/fast/", 
      "term": "Jensen, Smoke (Fictitious character)", 
      "weight": 0.0266
    }, 
    {
      "scheme": "http://id.worldcat.org/fast/", 
      "term": "Western fiction", 
      "weight": 0.0044
    }, 
  ], 
  "genres": [
    {
      "name": "Western", 
      "scheme": "http://librarysimplified.org/terms/genres/Simplified/", 
      "term": 254, 
      "weight": 1
    }
  ], 

  "customlists": [
    {
      "featured": false, 
      "first_appearance": 1413545710, 
      "list_id": 86
    }, 
  ], 

  "licensepools": [
    {
      "availability_time": 1423691583, 
      "available": true, 
      "collection_id": 1, 
      "data_source_id": 2, 
      "licensed": true, 
      "licensepool_id": 196028, 
      "medium": "Book", 
      "open_access": false, 
      "quality": 0.743, 
      "suppressed": false
    }
  ]
}
```

The Elasticsearch index is full of documents like this. We fill up the index ahead of time, and when we need to handle a search for `william johnstone idaho`, or list all the books in the `Mountain Man` series, we build an Elasticsearch query that tells us exactly which works should go into our OPDS feed.

# Generating the documents

Where do these big JSON documents come from? They're generated by the `Work.to_search_documents` class method in [core/model/work.py](https://github.com/NYPL-Simplified/server_core/blob/master/model/work.py).

This method is really complicated, so let's focus on the database join it creates (which is also pretty complicated). Here it is diagrammed out.

```
Work
|
+--Edition
|  |
|  +-Contribution--Contributor
|  |
|  +-Identifier
|    |
|    +-Classification--Subject
|
+--WorkGenre--Genre
|
+--CustomList
|
+--LicensePool
```

From top to bottom, this means that for every work, we want to find:

* Information that comes from the `Work` object itself, such as its database ID
  and fiction status.

* Bibliographic information about the work -- title, medium (`Book` or
  `Audio`), series name, numeric position within the series, and so
  on. This comes from a special `Edition` associated with the `Work`
  object, called the "presentation edition"

* Information about the book's `Contributor`s -- its author and anyone
  else who worked on it, such as William W. Johnstone. This
  information is associated with the presentation edition, not
  directly with the `Work`. We'll use this to make feeds of works by a
  specific author or audiobook narrator.

* Information about the subject-matter `Classification`s associated
  with the work, such as `Jensen, Smoke (Fictitious character)`. This
  information comes from sources like OCLC, and it's associated with
  an Identifier associated with the presentation edition (such as an
  ISBN). We don't show this information directly to library patrons,
  because it's not in any consistent format, but it's useful for handling
  searches like `etiquette` or `vegan cooking`.

* Information about the `Genre`s under which the work has been
  classified -- Western, Biography, and so on. By the time we get
  here, genre classifications have already been derived from the
  subject matter classifications, through a process that turns that
  miscellaneous information into a relatively small number of sections
  like you'd find in a bookstore or a branch library. This lets us
  handle searches that combine general terms with specific terms, such
  as `science fiction aliens`.

* Information about the `CustomList`s to which a librarian has
  manually added this book, or to which an external service (such the
  the New York Times best-seller list) has automatically added this
  book. This lets us make feeds of books found in those lists.

* Information about the `LicensePool`s that allow patrons to actually
  get copies of this book. We don't need detailed availability
  information about every `LicensePool` -- if we stored that in
  Elasticsearch, we'd have to update a books' search index every time
  someone borrowed it or put it on hold. But we do need to know
  whether a book is currently available, so that we can make feeds
  that only show currently available books. We need to know which
  collection provides each `LicensePool`, so that we don't show people
  books that are held by a different library. And we need to know the
  time each book was added to its collection, so we can generate feeds
  that put new acquisitions at the top.

`Work.to_search_documents` creates a big SQL query that grabs this information for up to 500 works at a time. It uses special Postgres functions -- `row_to_json`, `array_to_json`, and `array_agg` -- to process the data inside the database. The upshot is that this SQL query doesn't return normal data model objects. It returns JSON: one JSON object for each work matched by the query. And the _format_ of these JSON objects is exactly the format in the example above, the format that Elasticsearch is expecting.

This is very complicated, but Python code to do the same thing would also be pretty complicated, and it would be about 100 times slower. Since it's so important to keep the Elasticsearch index up to date, it's worth a lot of effort to optimize the generation of documents that go into the index.

# Initializing the index

`Work.to_search_document` doesn't actually touch the Elasticsearch index -- it just runs a SQL query that generates a bunch of JSON objects. The code that actually touches the Elasticsearch index is kept in [core/external_search.py](https://github.com/NYPL-Simplified/server_core/blob/master/external_search.py), and that's where we're going to spend most of our time for the rest of this document.

Before we can use an Elasticsearch index, we must do four things:

1. Create a [mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) document, so that ElasticSearch knows what to do with the documents we provide. This is the responsibility of the `CurrentMapping` class.
2. Create the current version of the index (currently `circulation-works-v4`), based on the mapping document. This can be done with a single HTTP request, and it's the responsibility of the `ExternalSearchIndex.setup_index()` method.
3. Change the `circulation-works-current` alias to point to `circulation-works-v4`. When we actually run queries against the Elasticsearch index, we'll be using this alias.
4. Install server-side scripts. These scripts let us run algorithms on the Elasticsearch server that would be impractical to run on our side. Currently we have only one server-side script: `simplified.work_last_update.v4`, which calculates the 'last updated' value for a work in a specific context.

At this point we can start running `Work.to_search_document()` to get huge chunks of JSON, and start dumping the JSON into `ExternalSearchIndex.bulk_update()`. Before long, we'll have a fully populated search index.

However, two of these steps are quite complicated: the mapping document and the server-side scripts. To fully understand the system, we'll need to go into detail for both of these.

# The mapping document

The mapping document is a special JSON object -- different from the ones generated by `Work.to_search_document()` -- that tells Elasticsearch how to index the `Work.to_search_document()` objects. It's conceptually similar to a database schema.

The `CurrentMapping` class contains all the information that's specific to the current version of the mapping document. The `Mapping` and `MappingDocument` classes help generate the JSON in the format Elasticsearch expects, but the important information is almost all in `CurrentMapping`.

## Query vs. Filter Context

Before we get started, there are a couple Elasticsearch concepts I'm going to refer to over and over again.

* _Query context_: The input is a string of text such as `italian romance` or `raina telemger`. This text was written by a human being, most likely using a lousy mobile phone keyboard. We are trying to match this string against our entire collection of books to find the books that the human being is most likely looking for.

* _Filter context_: The input is a set of criteria (e.g. "YA science fiction books on the 'staff picks' list"). We are using these criteria to _eliminate_ large chunks of the search index, leaving only the books that match the criteria.

Query and filter context are not mutually exclusive. When a patron of library A asks for `italian romance`, the query is executed in query context. But we only want to show books that are actually in library A's collection. This means filtering out books from other collections, and that's done in filter context.

## Why do we need a mapping?

Intuitively, you'd expect a search for `law of the mountain man` or `mountain man` to rate our example book pretty highly. And if you just dump the example document into Elasticsearch without providing a mapping, Elasticsearch will take care of that for you. For basic stuff, the mapping is optional.

But a lot of advanced features of Elasticsearch _do_ require a mapping. Almost everything we do that runs in filter context needs to have a basis in the mapping document. All the fields we use as input into server-side scripts must have their data types defined in the mapping. Most of our text fields need to be indexed multiple times using different rules--_that_ requires a mapping, even though the resulting fields are only used in query context. And so on. It turns out that (but not all) of the fields you see in the example document need to be defined in the mapping document.

## Properties

Properties are the core of a mapping document. In the `CurrentMapping` constructor we define the data types for each property we need to map:

```
        fields_by_type = {
            "basic_text": ['title', 'subtitle', 'summary',
                           'classifications.term'],
            'filterable_text': ['series'],
            'boolean': ['presentation_ready'],
            'icu_collation_keyword': ['sort_author', 'sort_title'],
            'integer': ['series_position', 'work_id'],
            'long': ['last_update_time'],
        }
        self.add_properties(fields_by_type)
```

This is saying that Elasticsearch needs to know know that the `presentation_ready` property will always be a boolwan, the `work_id` property will always be an integer, and `series` will always be a "filterable_text" -- whatever that is. The details of the various data types are explained below.

There are a few properties, like `publisher`, which aren't present in the mapping. We use those fields, but not in a way that requires any special treatment from Elasticsearch.

## Subdocuments

In the example, some of the field names like `title` and `last_update` have normal values--strings or numbers. But some fields -- `contributors`, `classifications`, `genres`, `customlists`, and `licensepools` -- have values that are lists of other JSON objects.

These lists correspond to the database joins between `Work` and other tables: `Contributor`, `Subject`, `WorkGenre`, `CustomListEntry`, and `LicensePool`. One work can have many contributors, or be on many custom lists, so when we create an Elasticsearch document for a work, we include all of them, as a list of sub-documents.

We need to define the data types for fields from three of these subdocuments: `contributors`, `licensepools`, and `customlists`. Although we do searches and filters on `genres` and `classifications`, they're not involved in our use of any advanced Elasticsearch features, so it's not a requirement that we define the subdocuments.

### Contributors

We primarily use the `contributors` subdocument to create feeds of books by a particular author. We don't use it when sorting feeds by author; instead we use the `sort_author` field in the main document.

```
        contributors = self.subdocument("contributors")
        contributor_fields = {
            'filterable_text' : ['sort_name', 'display_name', 'family_name'],
            'keyword': ['role', 'lc', 'viaf'],
        }
        contributors.add_properties(contributor_fields)
```

### License pools

We primarily use the `licensepools` subdocument to filter out books that shouldn't be shown. There are many reasons why this might happen, but some big ones are:

* They're in some other library's collection.
* The library no longer owns any copies.
* There are no copies available, and the patron asked for books that are available now.
* They're audiobooks from a vendor whose audiobook API we don't support.

```
        licensepools = self.subdocument("licensepools")
        licensepool_fields = {
            'integer': ['collection_id', 'data_source_id'],
            'long': ['availability_time'],
            'boolean': ['available', 'open_access', 'suppressed', 'licensed'],
            'keyword': ['medium'],
        }
        licensepools.add_properties(licensepool_fields)
```

### Custom lists

We primarily use the `customlists` subdocument to create feeds of books that are on specific lists. If a library has a 'staff picks' lane, that lane is based on a custom list managed by library staff. The books in that list have a corresponding entry in their `customlists` subdocument. The 'staff picks' lane is generated by sending a query to ElasticSearch that only picks up books with an appropriate `customlists` entry.

```
        customlists = self.subdocument("customlists")
        customlist_fields = {
            'integer': ['list_id'],
            'long':  ['first_appearance'],
            'boolean': ['featured'],
        }
        customlists.add_properties(customlist_fields)
```

## Data types

Every one of these properties has a data type: `integer`, `filterable_text`, and so on. This part of the document explains what the types mean, without going into too much detail.

### Built-in data types

The data types `integer`, `long`, `boolean`, and `keyword` are defined by Elasticsearch. The first three mean what you think they mean; the only complicated ones are `keyword` and `icu_collation_keyword`.

As we'll see, text fields usually have some 'fuzz' that lets a query match a field even if the text doesn't quite match. For most fields, this 'fuzz' is good, although we are a little picky about exactly how the fuzz is applied in different scenarios. We want a search for `john le carre` to match "John le Carré". In a query context, we don't care about little things like capitalization or accents.

But in a filter context we _do_ care about the little things. Elasticsearch won't use `text` fields in a filter context. Text fields can be used in filter context, but only if they're indexed as `keyword`. With `keyword`, there is no 'fuzz' -- nothing but an exact match will count. If the value of a `keyword` value is "Primary Author", then only `Primary Author` will count as a match -- not `primary author`, not `Author`, not `Primary Äuthor`.

We mainly use `keyword` fields for sorting (e.g. sorting a list of books in a seires by series position). Since that kind of sorting happens in filter context, Elasticsearch won't sort on a normal text field. It has to be a numeric field (like `series_position`) or a keyword. Our custom `sort_author_keyword` and `filterable_text` types (defined below) are variants on the `keyword` data type.

As for `icu_collation_keyword`... it's the same as `keyword`, but it implements the [Unicode Collation Algorithm](https://unicode.org/reports/tr10/) to improve sorting. This is important when we're sorting on a textual field (such as the title of a book) rather than a numeric field (such as series position).

### `basic_text`

Let's think about `description`. It's a textual description of a book, like you would see on the back cover. If we simply told Elasticsearch to index this property as `text`, Elasticsearch would run the description through [the standard Elasticsearch analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html). The text would be converted to lowercase, so that a search for `idaho` would match "Idaho". The text would be split up into tokens, so that you could search for `hunted small army coffee bullets` instead of needing to put all the words in exactly the right order.

That's pretty good, but there are some situations where we need to do things differently. When we define `description` as `basic_text`, we're actually defining three different fields:

* `description.standard` : A `text` field analyzed using the standard Elasticsearch analyzer.
* `description.minimal` : A `text` field analyzed using the custom `en_minimal_analyzer`.
* `description` : A `text` field analyzed using the custom `en_analyzer`.

Elasticsearch will index the description three different times, using slightly different rules each time. When we come in with a search query, we'll be able to use the different sets of rules to test out different hypotheses about what the user is trying to search for. We'll pick the hypothesis that gets the highest score, and hopefully deliver the best possible search results.

The implementation of this "define three different fields" thing is in
`MappingDocument.basic_text_property_hook`. The `add_property` method
(called many times by `add_properties`) knows to check for a
`_property_hook` method named after the data type. If that method
exists, it's called once for every property of that type. This lets us
implement custom data types while keeping the `CurrentMapping`
constructor simple.

### `filterable_text`

A `filterable_text` field is the same as a `basic_text` field, except the value is indexed _four_ times. The fourth time, it's indexed as a `keyword`, not as a `text`. That is, the value is stored as-is without any processing.

This is necessary because ElasticSearch won't use a text field in filter context -- it has to be a keyword field. That's bad news for a field like `series`. We want to use `series` in query context, so that if someone searches for `babysitters club` we can find books in the series they're clearly looking for. But we also want to use `series` in a filter context, so we can build a list of _only_ the books in the `Baby-Sitters Club` series, excluding unrelated books that mention babysitters or clubs in their descriptions.

`filterable_text` lets us have it both ways. We set up `series` as a `filterable_text`, and `Baby-Sitters Club` gets indexed three different ways so it can be used in a query context, like a `basic_text`. But it _also_ gets indexed as-is, as a `keyword`, so it can be used in a filter context for filtering and sorting.

### `sort_author_keyword`

This is a slight variant on `icu_collation_keyword` which we use for the `sort_author` field when we're sorting a list of books alphabetically by author. The only difference is that we want some regular expressions to be applied `sort_author` before it's indexed. The normal `icu_collation_keyword` type doesn't allow us to do this, so we kind of had to reinvent that data type and also mix in this feature.

To do this, we created a custom analyzer called `en_sort_author_analyzer`.

## Custom analyzers

Above I mentioned three 'custom analyzers'. The `basic_text` and `filterable_text` data types make use of `en_analyzer` and `en_minimal_analyzer`. The `sort_author_keyword` data type makes use of `en_sort_author_analyzer`. I'll discuss these custom analyzers in detail now.

* `tokenizer`: The name of an algorithm for turning a string into a series of tokens. We normally use the `standard` tokenizer, which basically splits on whitespace, turning `"Law of the Mountain Man"` into `["Law", "of", "the", "Mountain", "Man"]`. The only other tokenizer we use is `keyword`, which doesn't do anything -- the string is used as-is.
* `char_filter`: A chain of simple transformations -- each no more complicated than a regular expression -- to apply to the text before it's tokenized. 
* `filter`: A chain of transformations to apply to each token, _after_ tokenization.

### `en_analyzer`

* For `tokenizer` we use `standard`.
* We provide one `char_filter`. It's called `html_strip`, and it removes HTML (such as the <p> tags in the `description` of the sample book document).
* For `filter` we choose four transformations:
** `lowercase` converts all tokens to lowercase.
** `asciifolding` converts accented characters to their ASCII equivalents.
** `en_stop_filter` removes English stopwords like "the".
** Our custom `en_stem_filter` configures Elasticsearch's [stemmer token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html) to perform stemming of English words. So "talking" would become "talk", "loved" would become "love", and so on.

Once everything is done, `Law of the Mountain Man` has become `["law", "mountain", "man"]`.

### `en_minimal_analyzer`

This analyzer is the same as `en_analyzer` except for the final `filter` in the chain. Instead of `en_stem_filter`, we use a second custom filter, `en_stem_minimal_filter`.

These two filters are almost exactly the same. They're both the Elasticsearch [stemmer token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html), but `en_stem_minimal_filter` is configured with a less aggressive English stemmer.

### `en_sort_author_analyzer`

This one's a little complicated. We'd really like to define `sort_author` as a standard `icu_collation_keyword` data type and call it a day. Unfortunately, we need one little feature that `icu_collation_keyword` doesn't support: the ability to specify a custom list of `char_filter`.

So we need to define a custom analyzer, `en_sort_author_analyzer`. It looks like this:

* The tokenizer is `keyword`, meaning that values aren't tokenized at all -- they're used as-is.
* The `filter` includes a custom filtere called `en_sortable_filter`, which recreates the effect of the `icu_collation_keyword` datatype.
* The `char_filter`--the reason we're doing this--is a chain of five [Java regular expressions](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-replace-charfilter.html).

What do these regular expressions do that's so important? They normalize variations in the way author's names are presented, so that authors group together better.

* The special author `[Unknown]`, which we use to indicate that we don't have any author information, is converted to the last legal Unicode character. This ensures that books where we don't have author information will always appear last in a list ordered by author, rather than sorting at the top or with the letter U
* If a work has multiple authors, everything except the primary author is removed.
* Parentheticals are removed, so "Wells, H.G. (Herbert George)" becomes "Wells, H.G.".
* Periods are removed, so "Wells, H. G. becomes "Wells, H G".
* Spaces are collapsed for people whose sort names end with initials. So "Wells, H G" becomes "Wells, HG".

The upshot of this is that all of these sort name are treated the same for sorting purposes.

* Tolkien, J. R. R.
* Tolkien, J. R. R. (John Ronald Reuel)
* Tolkien, J.R.R.
* Tolkien, JRR
* Tolkien, J R R
* Tolkien, J. R. R.; Tolkien, Christopher

This means that in a list of books ordered by author, all of Tolkien's works will be grouped together and ordered by title -- basically what you'd expect to see on a library bookshelf.

## The server-side script

The final thing we need to define isn't part of the mapping document at all. It's a script written in the [Painless](https://www.elastic.co/guide/en/elasticsearch/painless/current/index.html) scripting language. The source code to this script is defined in the Python class variable `CurrentMapping.WORK_LAST_UPDATE_SCRIPT`, and its job is to figure out the "last update time" for a given work.

Why do we need a script for this? After all, we keep `Work.last_update_time` updated whenever a work's metadata changes, and we store that value in the search index. The problem is that a single work can have different "last update times" in different contexts.

For example, let's say a book published five years ago suddenly makes it big and shows up on the best-seller list. If you're asking ElasticSearch for a list of all the books in a library's collection, this book is nothing new. Its "last update time" might be years ago. But if you're asking for a list of all the books on the best-seller list, the book's "last update time" is very recent. The concept of "last update time" needs to take into account the time the work was added _to a relevant list_.

Similarly, let's say a book has been in Library A's collection for five years. Then one day Library B buys a license for the same book. That has nothing to do with Library A -- the book should stay near the bottom of Library A's "last update" feed. But for Library B, this book is brand new, and it should show up at the top of Library B's "last update" feed. The "last update time" needs to take into account the time the work was added _to a relevant collection_.

That's why we need a server-side script. This value can't be calculated ahead of time: it depends on which combination of collections and which custom lists we're looking at. We can calculate this value inside the circulation manager, once we have the search results, but that won't help us _sort_ a list by this value. It's the Elasticsearch server's job to sort a list, and it can't sort by a value it has no way to calculate. We need to give Elasticsearch the ability to calculate this value, so that we can sort a feed with the "most recent" books first -- whatever that means in context.

# Filtering

Now we're ready to see how we use Elasticsearch to do the two jobs that the database is bad at. We're going to start with the job of quickly finding exactly which books belong in a given lane.

When you think of "Elasticsearch", you probably think of the other job -- processing search requests. I'm going to go through the "filter" job first, because the "search" job needs to use the filtering feature, _plus_ some extra stuff.

## The `Filter` constructor

The `Filter` class (in external_search.py) implements this entire feature. Its job is to take requirements defined in terms of our business model objects -- `Lane`, `CustomList`, `Contributor`, and so on -- and express them in terms of our Elasticsearch mapping document.

The constructor for `filter` takes a large number of arguments representing every currently supported reason to exclude a book from a feed. Some examples:

* We only want books in certain collections (so people don't see books that are in some other library.)
* We only want books in certain languages.
* We only want books in certain genres.
* We only want books that are on certain custom lists.

All of this data is stored in the `Filter` object under construction. Then, whenit's time to go to Elasticsearch, we call `Filter.build()`, which turns this information into a bunch of `Q` (for "query") objects from the `elasticsearch_dsl` library. These objects represent constraints on the Elasticsearch data set, similar to the constraints imposed by WHERE clauses on a SQL dataset. These query objects will be converted into JSON and sent over to the Elasticsearch server.

I won't go into detail on how every single aspect of `Filter` is translated to an `elasticsearch_dsl` query. Instead, I'll go through the different types of examples, so that you'll be able to read through the code and understand it.

### Faceting

The `Filter` constructor takes one argument `facets` which doesn't affect the Elasticsearch query in any predictable way. The `facets` object represents restrictions set by the patron at the other end of an HTTP request. For example, a patron may ask to see only books in the "Audiobooks" entry point, or to only show books that are currently available.

Once all the other work is done, the `Filter` constructor calls `facets.modify_search_filter()` on its faceting object. This method can add additional constraints on the `Filter` object, or modify anything that was set earlier in the constructor. For an example, see `SearchFacets` in [core/lane.py](https://github.com/NYPL-Simplified/server_core/blob/master/lane.py). Under certain circumstances it will modify the `media` and `languages` attributes of the `Filter` object.

Note that the faceting code never directly interacts with any Elasticsearch code. All it can do is modify data associated with the `Filter` object, so that when `Filter.build()` is called, it behaves differently. This keeps all the code that touches Elasticsearch in one file: `external_search.py`.

## `Filter.from_worklist`

Most actual `Filter` objects are created through `Filter.from_worklist`. That's because most real-world filters are trying to restrict Elasticsearch results to books that fit in a specific `Lane`.

Let's say we're getting a list of all the books in a library's English "Science Fiction" lane. That lane implies a number of restrictions:

* Books must be in English.
* Books must be in the "Science Fiction" genre, or one of its subgenres.
* Books must be aimed at an adult audience.
* Books must be in a collection associated with _this particular library_.

`Filter.from_worklist` takes a `WorkList` object such as a lane, and translates its restrictions into `Filter` restrictions. These restrictions can, then, be translated into Elasticsearch query objects that will match only the books that belong in the `WorkList`.

## `Filter.build`

Now it's time to see how the information associated with a `Filter` object -- whether it came in through the constructor or through `Filter.from_worklist` -- gets turned into real Elasticsearch queries.

Recall that our `Works` get indexed as JSON documents. The documents we'll be querying look like this:

```
{
  "_id": 122940, 
  "work_id": 122940,
  "presentation_ready": true,
  "title": "Law of the Mountain Man", 
  "medium": "Book", 
  "language": "eng", 
  "audience": "Adult", 

  "customlists": [
    {
      "featured": false, 
      "first_appearance": 1413545710, 
      "list_id": 86
    }, 
  ], 

  "licensepools": [
    {
      "availability_time": 1423691583, 
      "available": true, 
      "collection_id": 1, 
      "data_source_id": 2, 
      "licensed": true, 
      "licensepool_id": 196028, 
      "medium": "Book", 
      "open_access": false, 
      "quality": 0.743, 
      "suppressed": false
    }
  ]
}
```

Each document contains some basic information like `medium`, `language`, and `audience`. It also contains _subdocuments_ like `customlists` and `licensepools`. 

Elasticsearch has a variety of strategies for applying restrictions on a query to filter out JSON documents that don't match what we're looking for. The `elasticsearch_dsl` library provides Python classes that implement these strategies. The `Filter.build` method converts between the two worlds. We start with our version of "only find books aimed at adules" and we end with an Elasticsearch query that says the same thing.

### Main document filters

A filter on a field found in the main document, like `language` in the example above, must be handled differently from a filter on a subdocument, like `customlists` in the example above. The main document filters are simpler, so I'll cover those first.

Here's the bit of `Filter.build` which applies the filters for "titles in certain media" (such as audiobooks) and "titles in certain languages":

```
    if self.media:
        f = chain(f, Terms(medium=scrub_list(self.media)))

    if self.languages:
        f = chain(f, Terms(language=scrub_list(self.languages)))
```

* We're building the Elasticsearch filter in a variable called called `f`. Once it's complete, this object will be part of the return value from `Filter.build`.

* Every time we find out some new restriction on the Elasticsearch filter, we build a new query object (in this case, a `Terms` object) and connect it to the existing filter with a helper function. This is equivalent to connecting two SQL WHERE clauses with "AND".

* We build different query objects in different circumstances. When it comes to filters, you'll primarily see `Terms`, `Term`, and `Bool`. All of these classes come from the `elasticsearch_dsl.query` module, and each corresponds to a query type in the [Elasticsearch query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html). Although these queries can be combined in complicated ways, each individual query is pretty simple. The `Term` class corresponds to the [term](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) query, which matches a record if its value for an attribute is one specific value. The `Terms` class corresponds to a [terms](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html) query, which matches a record if its value for an attribute is found in a _list_ of values.

* `scrub_list` is another helper function which turns a single item (such as `"eng"`) into a list (`['eng']`), but leaves a list alone. Since the value of `languages` is guaranteed to be a list, we can always use `Terms` to build our query, rather than having to decide between `Terms` and `Term`.

To sum up, we make a `Terms` query to restrict results to books that are in one of the selected languages, and another `Terms` query to restrict results to books with one of the selected media. We use `scrub_list` so that it doesn't matter whether a single language or a list of languages was provided. We use `chain` to ensure that when there are multiple restrictions, all restrictions are enforced at once.

At the end of `Filter.build` we have a single Elasticsearch query object. Maybe it's big, with many restrictions, like a SQL WHERE clause with many ANDs in it. Maybe it's small and matches almost everything in the search index. Either way, it represents all of the active restrictions this `Filter` object puts _on the main search document_. But it doesn't represent everything. We still have to deal with restrictions on the sub-documents.

### The nested filters

Restrictions on the sub-documents, like `customlists` and `licensepools`, must be handled differently. We handle them using a dictionary called `nested_filters`. This dictionary keeps track of a _list_ of Elasticsearch filters for each sub-document.

Here's an example: we're looking at `Filter.collection_ids`, which restricts the query to books that are in specific collections. This is how we avoid showing patrons of Library A, books that are only available in Library B.

```
    collection_ids = filter_ids(self.collection_ids)
    if collection_ids:
        collection_match = Terms(
            **{'licensepools.collection_id' : collection_ids}
        )
        nested_filters['licensepools'].append(collection_match)
```

Again, the filter is expressed using Elasticsearch's `Terms` query object. This says that a book matches the filter if _any one_ of its `licensepools` subdocuments has a `collection_id` that's found in the provided list of `collection_ids`.

In the previous section, we used the `chain` helper function to add a bunch of queries onto the big query we're building. Unfortunately, this doesn't work for filters on sub-documents. Filters on sub-documents need to be handled specially, and it's too complicated to deal with here. So we don't deal with it here. We just make the `Terms` object and add it to `nested_filters['licensepools']`. Later on (this happens in `Query.build`), we'll convert all of the objects in `nested_filters` to something Elasticsearch can understand.

Here's an example that's a little more complicated. This is how we represent a library patron's desire to only see books that are currently available.

```
    open_access = Term(**{'licensepools.open_access' : True})
    if self.availability==FacetConstants.AVAILABLE_NOW:
        # Only open-access books and books with currently available
        # copies should be displayed.
        available = Term(**{'licensepools.available' : True})
        nested_filters['licensepools'].append(
            Bool(should=[open_access, available])
        )
```

There are two reasons why a book might be 'available': it might have a LicensePool that's open access (in which case licensepools.open_access will be True), or it might have a LicensePool with available copies (in which case `licensepools.available`, a boolean we calculated way back in `Work.to_search_document`, will be True).

We want to match books where _either_ of these two reasons is true, but Elasticsearch's `Term` query will only match one field. So we build _two_ `Term` objects -- one for `licensepools.open_access` and one for `licensepools.available` -- and then we combine them using the `elasticsearch-dsl` `Bool` class, which corresponds to Elasticsearch's [bool](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) query.

We put our two `Term` queries in the `should` slot of the `Bool` query. The resulting query will match if there's a match for _at least one_ of the `should` queries. If none of the `should` queries match, the book doesn't match.

A `Bool` query has other slots, like `must` and `must_not`, which can be used to implement logical operations other than "at least one of these must match". If you look through the source code you'll find `Bool` used in a variety of ways.

### The universal filters

Up to this point, all of the filters we've considered have been optional. Sometimes we want a language filter, sometimes we don't. Sometimes only want books that are available now, sometimes we don't care whether they're available or not.

But there are a few filters that are universal. We _never_ want to show books that aren't presentation-ready. We _never_ want to show books that were once in a library's collection but no longer have any licensed copies. Basic stuff like that.

These filters are defined in two special methods, `Filter.universal_base_filter` and `Filter.universal_nested_filters`. They work the same way as the 'base filter' and 'nested filters' you saw earlier. The universal base filter is a single Elasticsearch query object that gets combined with other queries using `chain`, and the universal nested filters are kept in a dictionary so that they can be combined later, in `Query.build`.

## Sorting

There's one argument to the `Filter` constructor that's different from the others, and that's `order`. This doesn't affect which books match the filter -- it affects how the results are sorted.

The default sort order is (sort_author, sort_title, work_id). Works are sorted alphabetically by author, as they would be on a library bookshelf. Any given author's works are sorted alphabetically by title. If there are two copies of the same book by the same author (which can happen if two collections have the same book), the 'tiebreaker' is the internal work ID.

When you change the sort order, you either rearrange these fields or add a new on to the beginning. When you tell `Filter` to sort by title, you're actually telling it to sort by (sort_title, sort_author, work_id). When two books have the same title, we have to have a backup plan, and the backup plan is to sort by author name. When you tell `Filter` to sort by series position, you're actually telling it to sort by (series_position, sort_title, sort_author, work_id).

For most sort orders, that's all you need to know. But there are two sort orders that are more complicated.

### `last_update_time`

Sorting by 'last update time' is complicated because 'last update time' isn't stored in Elasticsearch. It's a calculated value, determined at runtime by running the Painless script kept in `CurrentMapping.WORK_LAST_UPDATE_SCRIPT`. We need to tell Elasticsearch to use this script to calculate the sort value. This is a little bit of Elasticsearch magic kept in `Filter._last_update_time_order_by`.

### `licensepools.availability_time`

Sorting by `licensepools.availability_time` (the time a book was added to a collection) is complicated because we're sorting on a value that's kept in a subdocument. This opens up two problems:

1. There might be multiple values for the field we're sorting by. If a book has two LicensePools in different collections, which is "the" availability_time?
2. We might also be using `licensepools.availability` in our filter. If a book has two LicensePools in different collections, but one of those collections is being filtered out, then its `availability_time` shouldn't count.

We solve both of these these problems in `Filter._availability_time_sort_order`.

The first problem is pretty simple to solve. "The" availability time for a book is the _earliest_ time it was added to _any_ relevant collection. If you've already seen a book, it shouldn't show up as a 'new arrival' again just because the library got another license for it somewhere else.

We can tell Elasticsearch this is the rule by specifying `mode="min"` when defining the sort order. This is equivalent to the `MIN` function in this SQL statement:

```
select work_id, MIN(availability_time) from licensepools GROUP BY work_id;
```

The second problem is more complicated. We have to apply our `collection_id` filter twice: once in the part of the Elasticsearch query that decides which books match the filter (this happens in `Filter.build`), and _again_ in the part of the query that decides how to order the results.

## Custom scoring functions

At this point we have everything we need to generate big OPDS feeds of books from any `Lane` or `WorkList`, using any combination of facets specified by the client. But not all of the OPDS feeds we generate come from just one lane. When someone opens the SimplyE app, the first OPDS feed they see combines books from many different lanes into a grouped feed.

The books in this list aren't sorted by title or author or anything else. In fact we don't really want them to be 'sorted' at all. For every lane in the feed, we want a _random_ selection of high-quality works that belong to that lane and are currently available.

We can take care of 'books that belong to the lane' with the `Filter` code you've already seen. But how do we get a random selection of high-quality works?

Well, up to this point we've been using Elasticsearch's "sort" functionality to get books in a predictable order. But when you run a search against an Internet search engine, the results don't come back "sorted" in any recognizable sense. You expect the best matches to be at the top, and that's it.

Elasticsearch has its own algorithm for assigning a numeric score to each document that matches a search. To get a random selection of high-quality works, we just have to replace that algorithm with one that favors "high quality" and "randomness".

This is done in `Filter.featurability_scoring_functions`. We use a variety of Elasticsearch tricks to define different ways of calculating a numeric score for a work:

* A high `Work.quality` score counts for a lot, up to a point.
* Being currently available counts for a lot.
* Being featured on a relevant `CustomList` is _huge_.
* Random chance counts for a little bit.

The `featureability_scoring_functions` method returns a list of these tricks. `ExternalSearchindex.query_works()` (the code that actually runs an Elasticsearch query) tells Elasticsearch to try every one of these tricks and add up all the scores, using `score_mode="sum"`. When the query runs, the works on the first page of results will be the highest-scoring works according to this algorithm. That gives us a random selection of high-quality works, exactly what we were looking for.

# Searching

Now we're ready to talk about the second job: handling search requests. We need to take a string typed by a human, a string like `science fiction aliens` or `diary of a stinky kid`, and find the books that are most likely to make that person happy. The first job -- finding all the books in a lane -- we could technically do with the database; it would just be really slow. But this really is a job that only a search engine like Elasticsearch can do. 

The key to solving this problem is the same "scoring function" idea we use to get a random selection of high-quality works. But instead of providing our own scoring function, we're going to exploit Elasticsearch's default scoring function.

The default scoring function is basically that books which match the search string get better scores than books that don't. But what does it mean to "match the search string"? There's no book called _Diary of a Stinky Kid_ -- whoever typed that in probably meant _Diary of a Wimpy Kid_. So we probably want to send _Diary of a Wimpy Kid_ as one of the search results. That book is part of a series -- should we send other books in the series, even though they have different titles? How important is the word "Stinky"? Maybe the person who typed in `stinky` was mixing up two different childrens' series. Can we find that other series and return its books as well? Of all the books we might send, which ones should we send first?

How about `science fiction aliens`? There's no book with that title, and the person who typed that probably doesn't want one. They're looking for a certain _type_ of book -- science fiction novels that feature aliens. A novel like _Rendezvous with Rama_ is a better match for this search query than a book of literary criticism called _100 Years of Aliens in Science Fiction_, even though the book of literary criticism has all of the search terms in its title.

Elasticsearch doesn't know anything about books or how people search for books. We gave it a bunch of JSON documents, it indexed them, and it's ready to search them. It's our job to tell Elasticsearch what a "good" match for a given search query might look like. It's our job to turn `science fiction aliens` into one type of query and `diary of a stinky kid` into another type of query.

This work happens in the `Query` object, found in `core/external_search.py`.

## Hypotheses

Someone who types in a search request might have meant any of a number of things. We handle this by forming _hypotheses_ about what the person might have meant. Then we tell Elasticsearch to test all of the hypotheses simultaneously.

Every book in the collection is given a score for every hypothesis we provided, and Elasticsearch chooses the best score for each book -- the most generous interpretation possible of why someone who searched for `diary of a stinky kid` is actually looking for _Modern Warfare, Intelligence and Deterrence_. Then the books with the best scores overall -- which will be more on the _Diary of a Wimpy Kid_ end than the _Modern Warfare_ end -- are chosen as the search results.

We build the list of hypotheses in `Query.query`. Each hypothesis is a query object that would return search results if we sent it to Elasticsearch. Instead of doing that, though, we combine the hypotheses into a single `DisMax` query in `Query._combine_hypotheses`. The `DisMax` query is what tells Elasticsearch to try every hypothesis and pick the best one for each book.

Each hypothesis has a weight associated with it. We determined these weights through trial and error, but the general relative weights should make intuitive sense. If your search string is an exact match for the title of a book, then that book really should be the first search result. We ensure this happens by weighting the "exact match" hypothesis very highly.

The 'hypothesis' metaphor makes the search code modular. We can come up with a lot of strategies, and tell Elasticsearch to try all of them at once and see what works best for this particular search. Here are the strategies we've come up with so far:

### Simple query string match

First, we try a [simple query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-simple-query-string-query.html) against title, subtitle, series, summary, classification, primary author, publisher, and imprint. This is the closest thing we do to a generic Elasticsearch query. This query works realy well when a search string combines multiple types of information, like `the demon haunted world carl sagan` or `goldfinch novel`.

This query runs against fields that have been put through the standard analyzer. This removes English stopwords like 'and', and applies a stemming algorithm. This ensures that a search for `awaken` will find a book called _The Awakening_ -- you don't have to literally type in `the awakening`.

### Phrase match

We also try a [match phrase query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html) against title, author, or series. We use this to handle the most common type of search request: someone typed in a book title, author, or series, more or less verbatim.

For this query we use the `.minimal` variant of `title` and `series` -- the one we set up to be analyzed using our custom 'en_minimal_analyzer'. The main difference is this analyzer is less aggressive about stemming.

Let's say there are two books in the collection, _Awakened_ and _The Awakening_. If you search for `awakening`, the "simple query string" hypothesis will give both books the same score, because its analyzer converts all three strings to the base string "awaken". The less aggressive stemmer used by the 'en_minimal_analyzer' turns "Awakened" into "awakened" and "The Awakening" into "awakening". This hypothesis will weigh _The Awakening_ much higher than _Awakened_, because it can tell that _The Awakening_ is closer to what you actually typed in.

If you search for `awaken`, then the first hypothesis makes more sense and both books are equally good matches. But if you search for `awakening`, this hypothesis makes more sense and _The Awakening_ should show up first.

### Exact match

This is a _second_ match phrase query, against title and author only, which greatly boosts an exact match. If you _do_ literally type in `the awakening`, this pretty much guarantees _The Awakening_ will be the first result.

### Fuzzy match

This is a [multi match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html) against any combination of title, subtitle, series, summary, author, publisher, or imprint. This is similar to our first hypothesis, the simple query string query (which is itself a multi match query under the hood). The difference is that this handles the case where the search request contains typos or misspellings. This is where we try to get Elasticsearch to figure out that `raina telemger` refers to author Raina Telgemeier.

### Simple query match with filter

Finally, we have a hypothesis that the search term might include structured data that can be parsed out and turn into a filter. We can extract structured data of the following types:

* A fiction status: `asteroids nonfiction`
* A target age or grade level: `grade 5 dogs`, `age 10 and up`
* A target audience: `young adult divorce`
* A genre: `romance billionare`, `robert moses biography`

The query parser is a class called `QueryParser`. It doesn't have any Elasticsearch code in it, just string processing. The goal is to identify information that we understand, and _remove_ it from the query string so it doesn't send Elasticsearch down the wrong path.

Any identifiable information in the query string becomes one or more filters on the work fields: `fiction`, `target_age`, `audience`, and/or `genres.term`. That part of the query runs in filter context. The rest of the query string, the part we don't understand, becomes a "simple query string" type query, similar to the one used in the first hypothesis. This part of the query runs in query context.

So, `asteroids nonfiction` becomes a "simple query string" query against `asteroids` with a filter restricting it to nonfiction. `romance billionare` becomes a "simple query string" against `billionare` with a filter restricting it to the "Romance" genre. `age 10 and up` becomes an empty query that gives the same score to all children's books for age 10 and up, and ignores every other book.

Remember that this is just one hypothesis among many. A search for `modern romance` will find books in the "Romance" genre that have `modern` in one of their other fields, such as _Ginger's Heart: Modern Fairytale Series, Book 3_. But `modern romance` is also an exact title match for a book called _Modern Romance_, and the exact title match is probably going to show up first.