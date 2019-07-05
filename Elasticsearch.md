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
  because it's highly unstructured, but it's useful for handling
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


# The server-side scripts