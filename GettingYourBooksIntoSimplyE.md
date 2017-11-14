You're on this page because you have some books that you want to get into SimplyE. This can mean a
lot of different things, and this document will guide you through the
process.

The basic answer is: "set up an OPDS server that gives access to your
catalog". But you'll need to answer two questions to figure out what
features the OPDS server needs to have and what design you should use.

# For authors

If you're the author of a book, and you want to get the book into SimplyE, you should know that SimplyE is not a bookstore or a library. It's a mobile interface to a large number of libraries. There are no books in "SimplyE" per se; all the books are in one library or another. To get your book into SimplyE, you'll need to get it into one of these libraries. At that point it will be available to patrons of that library.

Your best bet is to upload your book to the [Internet Archive](https://archive.org/) or to get it onto [unglue.it](https://unglue.it/). The Internet Archive will be publishing its books through SimplyE in the near future, so this will make your book available directly through SimplyE. Many libraries in the SimplyE system use unglue.it as a source of free content, so getting your book into unglue.it will make your book available indirectly.

The rest of this document assumes that "you", the person who has books they want to get into SimplyE, represent a publisher, distributor, university, public library, or similar institution. In other words, we assume you represent an organization that deals with hundreds or thousands of titles--so many titles that it makes sense to set up your own OPDS server.

# Do you want to be in the library registry?

The first thing a user sees when they open the SimplyE mobile
application is the library registry, a searchable list of OPDS
servers. tries to match a person with an institution

If you want individual people to be able to find your OPDS server,
then you need an entry in the library registry. If your goal is to
distribute books through libraries or similar institutions, then you
don't need an entry in the library registry.

Some organizations that could/will/do show up in the library registry:

* The New York Public Library
* [Canadian Electronic Library](http://www.canadianelectroniclibrary.ca/)
* [The Internet Archive](https://bookserver.archive.org/catalog/)
* [BARD](https://nlsbard.loc.gov/login//NLS)

Some organizations that publish (or could publish) OPDS feeds but
won't show up in the library registry because they don't deal directly
with individuals:

* [Plympton](http://plympton.com/)
* [BiblioBoard](https://biblioboard.com/)

# Do you allow anonymous access?

Some OPDS servers are open for anonymous access by the general
public. Others restrict access to people with a user account. It may
be easy or difficult to get a user account; the distinction here is
whether a user account is required at all.

# Four types of OPDS servers

Once you make these two decisions, you can know approximately what
features your server will need to have.



| *Library registry?* | *Anonymous access?* | OPDS feeds | Authentication For OPDS | Search service | Authentication guard | Token service | Example |
| ------------------- | ------------------- | ---------- | ----------------------- | -------------- | - | - | - |
| Yes                 | Yes                 | X          | X                       | X              |                      |               | [The SimplyE Collection](https://instantclassics.librarysimplified.org/index.xml) |
| Yes                 | No                  | X          | X                       | X              | X                    |               | [NYPL](https://circulation.librarysimplified.org/) |
| No                  | Yes                 | X          | Optional                |                |                      |               | [Standard Ebooks](https://standardebooks.org/opds/all) |
| No                  | No                  | X          | X                       |                | X                    | X             | [Plympton](http://plympton.com/) |

The features are discussed below.

# OPDS feeds

[OPDS](http://opds-spec.org/specs/opds-catalog-1-1-20110627/) is the
standard at the core of SimplyE. An OPDS feed is a list of books. Each
entry in the feed contains metadata for a book (title, author, etc.)
and instructions on how to get a copy of the book.

One OPDS feed can link to another, just like one HTML page can link to
another. There are all kinds of reasons why one OPDS feed might link
to another, but these are the two most important, and should give you
the general idea:

* Division: Putting the fiction and nonfiction books in separate sections instead of mixing them together.
* Pagination: Putting 50,000 books on 100 small pages, rather than all on one huge page.

## OPDS as user interface

When the SimplyE mobile app is pointed at an OPDS server, the links
between OPDS feeds determine the user interface shown to SimplyE, just
as the links between HTML pages become the user interface of a
website. There are two main features of OPDS which elevate it past
just a big scrolling list of books:

* Grouped feeds, which create a Netflix-style browsing interface.
* Navigation feeds, which let a reader drill down into categories and subcategories of books.

(Note that the SimplyE mobile app doesn't actually support navigation
feeds yet. Grouped feeds offer a nicer version of the same
functionality, but it's more work on your end.)

If your OPDS feed is intended for distributing books through other
institutions, rather than for direct consumption through SimplyE, you
should probably just create a simple paginated list of books with the
newest books at the top.

## Static generation

In most cases, it's preferable to generate your OPDS feeds statically
ahead of time. As your collection becomes more complicated, it may be
more practical to generate OPDS feeds as needed and cache them.

## Common OPDS pitfalls

To get your content into a SimplyE circulation manager you'll need to include some information that's not strictly required by the OPDS spec.

### Author sort names

OPDS requires that you identify authors by name, as their name would appear on the front of a book:

```
<author>
 <name>Carla T. Robot</name>
</author>
```

SimplyE further requires that authors be identified by their name as it would appear in a card catalog:

```
<author>
 <name>Carla T. Robot</name>
 <simplified:sort_name>Robot, Carla T.</name>
</author>
```

Be sure to also define the `simplified` namespace in your `<feed>` tag along with the `opds` namespace and any others you're using.

```
<feed ... xmlns:simplified="http://librarysimplified.org/terms/" ...>
```

### Categories

It's legal to publish an OPDS entry without any `<category>` tags, but it'll be very difficult for people to locate that title in a catalog. To make sure your titles show up in lists of "Science Fiction" and "Biography", you should add some `<category>` tags that tell the circulation managers how to file the books.

The most common classification scheme is BISAC. Here's how an OPDS feed can indicate that a title should be classified under BISAC FIC02800 ("FICTION / Science Fiction / General"):

```
<category term="FIC028000" scheme="http://www.bisg.org/standards/bisac_subject/"/>
```

# Authentication For OPDS

An [Authentication For
OPDS](https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw)
document explains what it takes for a person to actually get access to
your collection. You can advertise three authentication techniques in
your Authentication for OPDS document:
 
* Anonymous access (no authentication)
* HTTP Basic Auth (username/password)
* OAuth client credentials grant (used only by the token service below)

We also support a wrapper around OAuth's authorization code grant. For
a working example, see Clever login in the Open Ebooks app. However,
the situation there is so unusual that we don't want to give general
advice. If you authenticate your users with an OAuth authorization
code grant, contact us and we'll figure out how to work you into this
strategy.

## Collection metadata extensions

The library registry and the SimplyE mobile client also recognize a
large number of
[extensions](https://github.com/NYPL-Simplified/Simplified/wiki/Authentication-For-OPDS-Extensions)
to Authentication For OPDS which let you specify detailed information
about your collection: things like a description, the audiences and
geographic locations you serve, the estimated number of titles you
have in various languages, and even the color scheme to use when
displaying your collection.

The automatic library registration process (not yet designed) takes
your Authentication For OPDS document as its input. These extensions
contain the information it needs--description, geographic locations,
and so on--to put your OPDS server in the library registry and help
people find it.

## Anonymous access

Even if your OPDS server allows anonymous access, you can't join the
SimplyE library registry without an Authentication For OPDS
document. There are two reasons for this. First, you need to
explicitly state that your OPDS server allows anonymous access, and
the Authentication For OPDS document is the place to do that. Second,
the library registry needs all that extension information--your
colleciton's description, audiences and languages, and so on--and that
stuff goes in the Authentication for OPDS document.

## Static generation

In most cases the Authentication For OPDS document can be a static
document. You only need to regenerate it if something about your
server's code or configuration changes.

# Search service

This is not technically required, but unless your collection is very
small, readers will expect to be able to search it. If you don't deal
directly with readers, then you probably don't need this feature.

## Opensearch

You can implement your search service however you want. The SimplyE
Collection implements it with an Amazon Lambda service; the
circulation manager uses a wrapper around an ElasticSearch. The only
restrictions are:

* Your search service must be advertised with an
[OpenSearch](http://www.opensearch.org/Home) document.

* Your search service must serve its search results in the form of an
OPDS feed.

You advertise your search service by linking to the OpenSearch
document from your OPDS feeds. Here's an example from The SimplyE
collection:

```
<link href="https://instantclassics.librarysimplified.org/search" type="application/opensearchdescription+xml" rel="search"/>
```

[Fetch that document](https://instantclassics.librarysimplified.org/search) and you'll see that it explains how to construct a URL that performs a search:

```
<?xml version="1.0" encoding="UTF-8"?>
 <OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
   <ShortName>Search</ShortName>
   <Description>Search</Description>
   <Tags></Tags>
   <Url type="application/atom+xml;profile=opds-catalog" template="https://instantclassics.librarysimplified.org/search?q={searchTerms}"/>
 </OpenSearchDescription>
```

[Build a search
URL](https://instantclassics.librarysimplified.org/search?q=dinosaur),
send a GET request to it, and you'll get an OPDS Feed of search
results.

# Authentication guard

Unless you allow anonymous access to your collection, you'll need to
have some kind of software that guards access to your books, and
possibly even to the OPDS feeds themselves. There are a variety of
ways of doing this, but most of the time this means implementing HTTP
Basic Auth and looking up credentials in some kind of data store.

# Bearer token propagation

[Bearer token propagation](BearerTokenPropagation) is only necessary in one case:

* You are a distributor who sells access to your collection to libraries.
* But you don't want the libraries downloading your books, rehosting them, and serving them to their patrons.
* Instead, you want the library patrons to come to you whenever they want to download a book.

To see the problem here, consider a specific case. I come to your site
and say "I'm a patron of library X; give me this book." You've agreed
to hand out copies of this book to patrons of library X, but _how do
you know I'm really a patron of library X?_ Am I supposed to show you
my library card? How do you know it's legitimate?

When you set up the contract with library X, you sent out a username
and password for that library to use when accessing your OPDS
server. If I, the patron, had that username and password, I could
access your OPDS server and download the book. But I shouldn't be
given that username and password. I'm not library X, I'm just a
_patron_ of library X.

Library X shouldn't have to give out its credentials to every patron
who asks. It should be able to delegate its authority to a patron for
purposes of downloading a specific book from your server. This is what
[bearer token propagation](BearerTokenPropagation) is for, and it's covered in a separate document.