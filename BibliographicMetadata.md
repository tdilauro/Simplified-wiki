# Vocabularies

Let's define a vocabulary as an agreed-upon set of real-world
semantics for otherwise meaningless strings. On its own, the word
"date" is ambiguous. It could refer to an object, an event, or a point
in time. To a computer, "date" is just a four-character string. It
doesn't mean anything at all. But in the Dublin Core vocabulary,
"date" has a precise meaning: it always refers to a point in time.

Two systems can only communicate with each other if they share a
vocabulary. Most vocabularies are ad hoc custom vocabularies designed
for one specific system. For instance, the 3M API uses terms like
"EventStartDateInUTC" whose meanings are defined only in the
documentation for the 3M API. (And sometimes not even then--terms like
"PhysicalISBN" are never formally defined.)

There are many, many vocabularies for describing books. It's probably
the most common type of vocabulary, after vocabularies for describing
people. For books, our problem is an abundance of *standardized*
vocabularies on top of the abundance of ad hoc vocabularies.

Some vocabularies are tied to a format (ONIX, Atom). Some are tied not
only to a format but to a specific piece of software (OverDrive,
3M). Some are format-neutral (Dublin Core,
http://schema.org/Book). But each vocabulary has an ideology: it
encapsulates the language used by a particular relationship.

For example, http://schema.org/Book encapsulates the language a
webmaster uses when talking a book to a search engine. ONIX
encapsulates the language a publisher uses when talking about a book
to a bookstore. They are both talking about books, but at different
levels of detail and for different purposes.

When deciding which vocabulary or vocabularies to support we need to
start with the question: who are we communicating with? What is our
relationship with them?

# Vocabularies that are tied to format

## MARC

MARC is the LoC's famous heavyweight vocabulary for bibliographic
information. There is a binary serialization and [an XML
serialization](http://www.loc.gov/standards/marcxml/). Both use
concise terms like "100" and "x" rather than human-readable terms like
"author" and "subcategory"; these form MARC's "content designation".

Project Gutenberg serves MARC records for its books, generated from
its RDF catalog.

The 3M API will serve an XML MARC record for purchased ebooks ("Get
MARC"). I don't think Overdrive offers this feature.

## ONIX

ONIX is "intended to support computer-to-computer communication
between parties involved in creating, distributing, licensing or
otherwise making available intellectual property in published form,
whether physical or digital." It is a heavyweight format like MARC,
but MARC is optimized for library catalog and ONIX seems more general,
intended for business-to-business use.

ONIX is an XML vocabulary. It seems to define both MARC-like
abbreviated terms like "b203" and equivalent human-readable terms like
"TitleText".

* [Introduction](http://www.editeur.org/files/ONIX%203/Introduction_to_ONIX_for_Books_3.0.pdf)
* [Spec downloads](http://www.editeur.org/93/Release-3.0-Downloads/)

## Atom

Atom is a media type based on XML ("application/atom+xml") originally
designed for serializing blog posts. It defines basic, generic terms
like "author", "category", "published", and "title".

Although spartan on its own, Atom is extensible through a "profile"
mechanism, and has been widely extended. Notably by OPDS, which
defines an Atom profile for catalogs of ebooks.

* [Definition](https://tools.ietf.org/html/rfc4287)
* [OPDS spec](http://opds-spec.org/specs/opds-catalog-1-1-20110627/)

# Format-independent vocabularies

Most of these are RDF vocabularies, which can be serialized in a
number of forms, used in Linked Data applications, and included in a
number of other formats (because pretty much everything in an RDF
vocabulary is a URI).

## http://schema.org/Book

Schema.org vocabularies are RDF vocabularies that can also be
expressed in HTML 5 microdata. Its "Book" vocabulary defines basic
terms like "author", "illustrator", "isbn", "copyrightYear", and
"genre".

* [Definition](http://schema.org/Book)

## Dublin Core

A basic "vocabulary of fifteen properties for use in resource
description", first created in 1995. These include "subject",
"creator", and "title". Since 1995 many additional terms have been
added to Dublin Core, like "isReplacedBy" and "abstract".

Project Gutenberg uses [Dublin Core's
recommendation](http://dublincore.org/documents/dc-rdf/) for
presenting bibliographic information in RDF. This means a lot of DCMI
Metadata Terms and limited use of the [DCMI Abstract
Model](http://dublincore.org/documents/abstract-model/)'s "memberOf"
term.

* [The fifteen: Dublin Core Metadata Element Set](http://dublincore.org/documents/dces/)
* [Everything: DCMI Metadata Terms](http://dublincore.org/documents/dcmi-terms/)

## Miscellaneous

In addition to Dublin Core, Project Gutenberg's RDF documents also
include one element from Creative Commons (for licensing), plus a
custom Project Gutenberg vocabulary containing miscellaneous
information about authors, ebook numbers, and download locations.

# Vocabularies tied to specific pieces of software

## 3M XML

A vocabulary for describing 3M Cloud Library's catalog. It looks to
have been autogenerated from the names used in C# classes on 3M's
side. AFAICT, the terms are not explicitly defined anywhere--you're
supposed to look at the names and figure it out.

Documents are probably served as application/xml, but I don't know yet.

## OverDrive JSON

Overdrive's [Metadata
API](https://developer.overdrive.com/apis/metadata) serves JSON
documents that use custom terms like "isOwnedByCollections", "title",
"shortDescription", and "subject" to describe books. Other APIs also
use this vocabulary when describing books. AFAICT, the terms are not
explicitly defined anywhere.

Documents are probably served as application/json, but I don't know yet.

## BiblioCommons

Bibliocommons's [Titles
API](http://developer.bibliocommons.com/docs/titles_id) serves JSON
documents that use custom terms like "authors", "isbns", and
"title". Each term is explicitly defined in the API documentation.

Documents are served as application/json. The internal format of the JSON
documents is not explicitly defined.

# Classification schemes

A lot of schemes have been devised to classify books.

* [BISAC](https://www.bisg.org/complete-bisac-subject-headings-2013-edition) Sample: "POLITICAL SCIENCE / Public Policy / City Planning & Urban Development"

* [BIC](http://editeur.dyndns.org/bic_categories) Sample: "FKC" (Classic horror and ghost stories), child of "FK" (Horror and ghost stories), child of "F" (Fiction).

* [Dewey Decimal classification](http://dewey.info/) Sample: "188" (Stoic philosophy), child of "18" (Ancient, medieval & eastern philosophy), child of "1" (Philosophy & psychology)

* [Library of Congress classification](http://www.loc.gov/catdir/cpso/lcco/) Sample: "QE521-545" (Volcanoes and earthquakes), child of "QE" (Geology), child of "Q" (Science)

* [Library of Congress subject headings](http://www.loc.gov/aba/cataloging/subject/) Sample: "Fundraising cookbooks"

* Bookstores like Amazon have their own proprietary classifications, e.g. "Books > Arts & Photography > Architecture > Urban & Land Use Planning". These also show up as GoodReads "genres".

* "Author" and "Series" are not good classifications on their own, but they do help to group books together. Readers of fiction tend to make decisions about authors and series rather than individual books.

* Folksonomic classifications like tags divide up the space of books into books that have a certain feature and books that don't. These show up as GoodReads "shelves".

* Lists of books similarly divide the space of books into books on the list and books not on the list.