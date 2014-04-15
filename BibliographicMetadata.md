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

Documents are served as a custom media type, application/vnd.overdrive.api+json, but I don't think the media type is formally defined anywhere.

## BiblioCommons

Bibliocommons's [Titles
API](http://developer.bibliocommons.com/docs/titles_id) serves JSON
documents that use custom terms like "authors", "isbns", and
"title". Each term is explicitly defined in the API documentation.

Documents are served as application/json. The internal format of the JSON
documents is not explicitly defined.

## GoodReads

XML format. Includes reviews.

## Amazon Product Advertising API

XML format. Includes reviews. I believe we have more favorable usage terms if we scrape the website as a spider rather than going through the API.

## LibraryThing

XML and JSON formats are provided under different licenses.

Includes lots of "Facts" about a book such as characters, important events, and quotations.

# Side-by-side comparison

## Vocabulary comparison

This table compares five major vocabularies for ebook sources, plus
one source for catalog data (BiblioCommons). It focuses on the terms we
care most about for identifying titles, tracking our inventory, and
downloading books once they've been checked out. We can add to this
table as we investigate more APIs.

<table border="1">
<tr>
 <th>Field</th> <th>Atom+OPDS</th> <th>Overdrive</th> <th>3M</th>
 <th>Axis 360</th> <th>Gutenberg</th> <th>BiblioCommons</th>
</tr>

<tr>
 <td>Internal ID</td>
 <td>atom:id or dc:identifier</td>
 <td>id</td>
 <td>ItemId</td>
 <td>titleId</td>
 <td>The URL that pgterms:ebook is rdf:about</td>
 <td>id</td>
</tr> 

<tr>
 <td>Permalink</td>
 <td>atom:id</td>
 <td>links[self], links[availability], links[metadata]</td>
 <td>BookLinkURL</td>
 <td>titleUrl</td>
 <td>Same as internal ID</td>
 <td>details_url</td>
</tr> 

<tr>
 <td>ISBN-13</td>
 <td>dc:identifier may be a urn:isbn: URI, but probably not</td>
 <td>metadata->formats[identifiers]</td>
 <td>ISBN13</td>
 <td>isbn</td>
 <td>-</td>
 <td>isbns</td>
</tr> 

<tr>
 <td>Title</td>
 <td>atom:title</td>
 <td>title</td>
 <td>Title</td>
 <td>productTitle</td>
 <td>dc:title</td>
 <td>title</td>
</tr> 

<tr>
 <td>Subtitle</td>
 <td>usually part of dc:title</td>
 <td>subtitle</td>
 <td>Subtitle</td>
 <td>-</td>
 <td>included as part of dc:title</td>
 <td>sub_title</td>
</tr> 

<tr>
 <td>Series</td>
 <td>- (some vendors may include with dc:title)</td>
 <td>series</td>
 <td>-</td>
 <td>series</td>
 <td>- (sometimes "Part II" etc. in dc:title)</td>
 <td>series</td>
</tr> 

<tr>
 <td>Author</td>
 <td>atom:author, atom:creator</td>
 <td>primaryCreator, metadata->creators</td>
 <td>Authors (list--separated how?)</td>
 <td>contributor (list--separated how?)</td>
 <td>dc:creator, with a pgterms:agent entry for each</td>
 <td>authors, additional_contributors</td>
</tr> 

<tr>
 <td>Description</td>
 <td>atom:summary (text only) or atom:content (usually HTML, often has buy form/download link and other misc metadata)</td>
 <td>metadata->shortDescription</td>
 <td>Description (HTML format)</td>
 <td>-</td>
 <td>-</td>
 <td>description</td>
</tr> 

<tr>
 <td>Publisher</td>
 <td>dc:publisher</td>
 <td>metadata->publisher</td>
 <td>Publisher</td>
 <td>publisher</td>
 <td>dc:publisher (it's always "Project Gutenberg")</td>
 <td>publishers</td>
</tr> 

<tr>
 <td>Imprint</td>
 <td>-</td>
 <td>metadata->imprint</td>
 <td>-</td>
 <td>imprint</td>
 <td>n/a</td>
 <td>-</td>
</tr>

<tr>
 <td>Publication date</td>
 <td>dc:issued (atom:published for when added to OPDS catalog)</td>
 <td>metadata->publishDate, metadata->publishDateText</td>
 <td>PubDate</td>
 <td>publicationDate</td>
 <td>dc:issued</td>
 <td>publication_date</td>
</tr> 

<tr>
 <td>Language</td>
 <td>dc:language</td>
 <td>metadata->languages</td>
 <td>Language</td>
 <td>language</td>
 <td>dc:language</td>
 <td>primary_language, languages</td>
</tr> 

<tr>
 <td>Reader Rating</td>
 <td>-</td>
 <td>metadata->starRating, metadata->popularity</td>
 <td>- (tracked, but not published through the API)</td>
 <td>-</td>
 <td>-</td>
 <td>-</td>
</tr> 

<tr>
 <td>Classifications</td>
 <td>atom:Category</td>
 <td>metadata->subjects, metadata->gradeLevels</td>
 <td>-</td>
 <td>subject, audience ("General Adult")</td>
 <td>dc:subject (LCC=Library of Congress Classification, LCSH=Library of Congress Subject Headings)</td>
 <td>suitabilities</td>
</tr> 

<tr>
 <td>Cover image</td>
 <td>link with rel="http://opds-spec.org/thumbnail"/"http://opds-spec.org/cover" /"x-stanza-cover-image"/"x-stanza-cover-image-thumbnail"</td>
 <td>images[thumbnail], metadata->images[cover] (contentreserve.com)</td>
 <td>CoverLinkURL (very high quality)</td>
 <td>-</td>
 <td>-</td>
 <td>-</td>
</tr> 

<tr>
 <td>Download link</td>
 <td>odps:indirectAcquisition for DRM-encrypted stuff. Otherwise link rel="http://opds-spec.org/acquisition" or "http://opds-spec.org/acquisition/open-access" or "http://opds-spec.org/acquisition/borrow" </td>
 <td>contentLink (using Download API)</td>
 <td>NO WAY</td>
 <td>downloadUrl from a 'checkout' call</td>
 <td>The rdf:resource of a dc:hasFormat tag</td>
 <td>n/a</td>
</tr> 

<tr>
 <td>Download link (sample)</td>
 <td>link rel="http://opds-spec.org/acquisition/sample" </td>
 <td>metadata->samples</td>
 <td>-</td>
 <td>-</td>
 <td>n/a</td>
 <td>n/a</td>
</tr> 

<tr>
 <td>Format</td>
 <td>'type' of acquisition link</td>
 <td>metadata->formats</td>
 <td>BookFormat</td>
 <td>availability/availableFormats</td>
 <td>dcam:hasFormat, each with a pgterms:file stanza</td>
 <td>n/a</td>
</tr> 

<tr>
 <td>File size</td>
 <td>-</td>
 <td>metadata->formats[fileSize]</td>
 <td>Size</td>
 <td>fileSize (but seems to only exist for audio books)</td>
 <td>pgterms:file/dc:extent</td>
 <td>n/a</td>
</tr> 

<tr>
 <td>Purchased copies</td>
 <td>-</td>
 <td>availability->copiesOwned</td>
 <td>TotalCopies</td>
 <td>availability/totalCopies</td>
 <td>n/a</td>
 <td>"copies" API for physical copies</td>
</tr> 

<tr>
 <td>Available copies</td>
 <td>-</td>
 <td>availability->copiesAvailable</td>
 <td>AvailableCopies</td>
 <td>availability/availableCopies</td>
 <td>n/a</td>
 <td>"copies" API for physical copies</td>
</tr> 

<tr>
 <td>Size of hold queue</td>
 <td>-</td>
 <td>availability->numberOfHolds</td>
 <td>OnHoldCount</td>
 <td>availability/holdQueueSize</td>
 <td>n/a</td>
 <td>n/a</td>
</tr> 

<tr>
 <td>Other</td>
 <td>atom:rights supersedes dc:rights. Library use case probably has to be worked out on a case-by-case basis.</td>
 <td>metadata->awards, metadata->reviews, metadata->popularity, metadata->sortTitle</td>
 <td>NumberOfPages (always seems to be missing or wrong), PhysicalISBN</td>
 <td>annotation ("ENGLISH"), addedDate (i.e. date added to inventory), minLoanPeriod, maxLoanPeriod, availability/updateDate, availability/Checkouts, availability/Holds
<p>Availability information per patron: holdQueuePosition, isInHoldQueue, isReserved, reservedEndDate, isCheckedout, checkoutFormat, checkoutStartDate, checkoutEndDate, downloadURl
</td>
 <td>dc:rights (some PG books are copyrighted and may have restrictions on distribution, but it's done on a per-book basis and spot checks turn up nothing special)</td>
 <td>pages, edition, contents (table of contents)</td>
</tr> 

</table>
