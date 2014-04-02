## APIs, SDKS etc..

* _Italicized bullet items_ are unresolved questions that need attention.

### Patron APIs

#### New NYPL patron

The current procedure for getting a NYPL card can be started online
(you get a temporary card ID that expires at the end of the day) but
completion requires going to a library, showing ID, and picking up a
physical card with a different ID on it. The PIN is not needed to
exchange a temporary card for a permanent card. I don't know whether
the temporary card ID is destroyed when the physical card is scanned.

We will be allowing anyone who boots up the app from a NYC GeoIP to
create a persistent card by providing their date of birth and
address. This requires an API for creating new, persistent NYPL
credentials.

The Brooklyn Public Library created a program called [My Library
NYC](http://mylibrarynyc.org/about), in which you can sign up online
for a unified library card that works with all NYC libraries. What API
are they using to create a NYPL credential?

The [http://techdocs.iii.com/patronws_patron_data.shtml](Innovative Millenium API) (user:pass nypl_s:chapter) includes a "Patron Update Web Service" which can [http://techdocs.iii.com/patronws_api_operations.shtml#createPatron](create a new patron).

#### Patron authentication

The [Innovative Patron
API](http://vendordocs.iii.com/patron/patronapi.shtml) (user:pass
nypl_s:chapter) can validate a patron based on identifier and PIN.

The [Innovative Millenium API](http://techdocs.iii.com/patronws_patron_data.shtml) (user:pass nypl_s:chapter) probably also does
validation, but I'm not sure how.

### Content

#### Gutenberg####

About 45,000 free public domain texts.

We plan to set up a mirror of the Gutenberg ePub documents. This gives us a text source over which we have complete technical and legal control.

* _Set up the mirror._
 - Provision a machine and copy over the ePubs.
 - Retrieve the [MARC records](http://gutenberg.readingroo.ms/cache/generated/feeds/) and insert an "Electronic resource" pointer in each. There are a number of sources for MARC records ([this one](http://www.gutenberg.org/feeds/catalog.marc.bz2) is from gutenberg.org but looks identical to the one at readingroo.ms), and [a third-party script](http://ebooks.adelaide.edu.au/meta/pg/) for converting Gutenberg RDF documents to MARC. 
 - Inject the MARC records into our catalogue.
 - Both the MARC records and the mirror should eventually be updated nightly. The Adelaide third-party library provides daily feeds of new and updated MARC records.
 - Come up with some way of identifying when a Gutenberg text is the same as an Overdrive text, and prefer the Gutenberg text.
* _How many books do we check out through Overdrive that we could replace with Gutenberg?_
* _Are there changes we could make to the default epubs that would improve user experience?_ Doing some manual work on the top 100 books would be worth it.

#### Internet Archive / Open Library ####

#### Overdrive

* [OverDrive APIs] (https://developer.overdrive.com/apis)
* [API for checking availability](https://developer.overdrive.com/apis/library-availability)
* [API for checking out a book and "locking in" a format (e.g. "ebook-epub-adobe") to get a download URL.](https://developer.overdrive.com/apis/checkouts)
* [API for downloading a book in the locked-in format.](http://developer.overdrive.com/apis/download)
 - _What format is the downloaded file? ODM? ASCM? We need to be able to open it in our reader._
* No API for returning a book early. ("If a format has been locked in, then users need to use software like OverDrive Media Console or Adobe Digital Editions to complete the return process.")
 - _We need to be able to do this from our reader._

#### 3M Cloud Library
3M's Cloud Library is accessible through license.  We will be executing a license as part of the development to begin integration work

#### HaithiTrust
Haithitrust Data API is a programmatic access layer to their catalogue repository.
* [Haithi DTrust Data API](http://www.hathitrust.org/data_api) Documentation in PDF format.
* We have access only to the public domain books that we volunteered to be scanned (about 40k volumes). _Is this accurate?_
* Full volumes are available for the Espressnet project only, and only in Espresso Book Machine format. (Search term: "Volume-type resources".) Everyone else is restricted to page images.
 - _We need access to full documents in ePub or PDF format._ 
* Google owns the OCR data and Hathi has a copy but cannot make it available to us.
 - _Can we get access to the data by making a deal with Google?_ James says there's no more than a 25% chance of this, but it's worth a shot.
 - _Can we do the OCR ourselves on a proactive or on-demand basis?_ David says this would be a big crowdsourcing project, but one that would be worth doing.
 - _Even if we can't have direct access to the OCR, can we do full-text search through a Hathi API as part of discovery?_

### O'Reilly Media

O'Reilly is a large technical publisher that is a pioneer in e-publishing, hostile towards DRM, interested in experiments, and with which Leonard has a good professional relationship. If we want to support publishers as ebook providers, O'Reilly would be a good place to start.

* [O'Reilly Media books on Overdrive](http://ebooks.nypl.org/2C44B93D-BE81-4C84-9D70-33ACCAD960E7/10/50/en/SearchResults.htm?SearchID=14666093s&SortBy=Relevancy)

### DRM

#### Adobe

The current standard for DRM. The only licit way to unlock a document protected by Adobe
DRM is to use Adobe's SDK ($$$). Adobe is trying to contribute a
binary module to Readium that will allow unlocking documents from
Readium, but both the format (binary) and the license (restrictive)
are unacceptable to the Readium Foundation.

* James expects this will eventually be resolved by Adobe releasing a binary module under a nonrestrictive license. _How is this going_?
* [Readium DRM interface documentation](https://github.com/NYPL/iOS-Reader/blob/master/Documents/Readium%20DRM%20interface.pdf?raw=true)

#### LCP

[Lightweight Content
Protection](http://idpf.org/epub-content-protection) is an open standard for DRM. We have [a draft of the spec](https://github.com/NYPL/iOS-Reader/blob/master/Documents/Lightweight%20Content%20Protection%20(LCP)%20Standard%201.0.pdf?raw=true) and it looks pretty easy to implement a client. The final standard should be released around May 2014.

* James is negotiating
with Overdrive and 3M to get them to serve us books protected with
LCP. _How is this going?_

### Catalogue

### BiblioCommons###

BiblioCommons contains the title listings for all of the ebooks we offer. It does not describe our _inventory_. BiblioCommons does not know how many copies of an electronic title we've bought, how many of those copies are currently lent out, or how many holds are on a title. All of that information is kept on Overdrive or 3M, and available via an API call for one particular title. (Thus the "check availability" button on BiblioCommons.)

BiblioCommons also does not support lifecycle events such as checkout. The only thing it knows is the URL to the Overdrive/3M page for a title.

Because of this, Dave believes that there is no point in integrating BiblioCommons with the app. Since BiblioCommons is how people will be checking out ebooks when they're not using the app, it may make sense to integrate any new book sources (such as Gutenberg) with BiblioCommons.

* [BiblioCommons] (http://developer.bibliocommons.com/blog/read/Welcome_to_the_BiblioCommons_API)
* [API documentation](http://developer.bibliocommons.com/docs)

### OPDS

[OPDS](http://opds-spec.org/specs/opds-catalog-1-1-20110627/) is an
Atom profile for describing catalogues of electronic publications. 

* Uses Atom's vocabulary for describing a book's metadata. Encourages the use of Dublin Core vocabulary except where Atom uses a different term for the same thing.
* Uses Atom feeds to describe lists of publications, which can be filtered or ordered according to facets. The concept of a facet is defined, but no particular facets are defined.
* Defines vocabulary for describing feeds of "new", "recommended", or "popular" items, as well as a user's "shelf" of acquired items.
* Uses OpenSearch as a search protocol.
* Defines a vocabulary for basic transitions (notably "borrow"), but does not define a protocol for carrying out those transitions.

According to (a comparison chart on Wikipedia)[http://en.wikipedia.org/wiki/OPDS#Comparison_of_OPDS_clients], a number of e-reading applications function as OPDS clients. Several bookstores (see below) offer OPDS feeds that can be presented to users of those applications, who can buy books from inside the application.

I think OPDS is a good place to start for a catalogue implementation. It's widely supported and it reinvents as little as possible--mostly bookstore/library vocabulary.
We might not want to serve Atom documents between the backend and the mobile client, since Atom is an XML format, but it shouldn't be hard to serve a (Siren)[https://github.com/kevinswiber/siren] version of an OPDS document. 

A number of other sites and publishers publish OPDS feeds of open access materials. We can use these to import items into our catalogue, or to create virtual catalogues of disparate materials.

For example, the simplest way to integrate our Gutenberg mirror into a larger catalogue might be to publish an OPDS feed for the Gutenberg mirror and having the larger catalogue sync with it daily. (We could even use this technique to keep the Gutenberg mirror in sync with Gutenberg, but I'm pretty sure Gutenberg would rather we use rsync.) A library that wanted to use our Gutenberg mirror could hook up its OPDS feed to their system.

Some OPDS feeds of open access materials:

* [Project Gutenberg](http://m.gutenberg.org/ebooks.opds/)
* [Internet Archive](http://bookserver.archive.org/catalog/)
* [Many Books](http://manybooks.net/opds/index.php)
* [Revues](http://bookserver.revues.org/?sort=OA) (journals, en français)
* [Ebooks Graruits](http://www.ebooksgratuits.com/opds/index.php)  (en français)
* [YouScribe](http://opds.youscribe.com/Catalog/books.xml) (en français)

Some OPDS feeds of books for sale:

* [Pragmatic Programmers](http://pragprog.com/catalog.opds)
* [O'Reilly Media](http://opds.oreilly.com/opds/)
* [FeedBooks](http://www.feedbooks.com/catalog.atom)
* [SmashWords](http://www.smashwords.com/lexcycle/)
* [All Romance Ebooks Online](http://www.allromanceebooks.com/epub-feed.xml)
* [BooksOnBoard](http://www.booksonboard.com/xml/catalog.atom)
* [Beam eBooks](http://stanza.beam-ebooks.de/) (in Deutsch)
* [Librusek](http://lib.rus.ec/new/opds) (в России)

### Lifecycle

Each content provider has its own workflow for checking out books. We have pretty much no visibility into this workflow, because all the important events happen on Overdrive or 3M's site. The result is we don't know what is currently in our inventory and what is checked out. 

Many features depend on this. For instance, queue management requires that we be able to understand the velocity of a license--how long it typically stays checked out.

* _We need near-real-time lifecycle updates from Overdrive and 3M._ We need this even once the app is launched, because people will continue to check out ebooks through BiblioCommons, creating lifecycle events that we didn't cause.
* The reader will trigger lifecycle events by calling a backend API which will dispatch to the content provider's API.

### Client libraries

[OAuth2.0 Client for iOS](https://github.com/AFNetworking/AFOAuth2Client) MIT License

#### Repositories from Readium Foundation
Readium SDK is an open source project developing a productized, high-performance, cross-platform rendering engine for EPUB 3 content, optimized for use in native applications (mobile/tablet and secondarily desktop systems). Simplistic test applications for Android, iOS, OS/X and Windows are part of the SDK, along with test frameworks. Readium SDK is extensible in various areas including DRM support. Readium SDK will be dual-licensed under both GPL and a commercial-friendly (non-copyleft) license. Note that the default license currently in the open source code on github is GPL.

A separate sub-project intends to in parallel develop an interoperable implementation of a lightweight content protection technology (DRM) that can be implemented into the modular framework of Readium SDK. This DRM will be available under a separate license, terms and conditions TBD.
* [Readium SDK](http://readium.github.io/sdk-api-doc/)
* [Readium Documents on Google Docs] (https://drive.google.com/a/nypl.org/?hl=en&pli=1#folders/0BzaNaBNAB6FjbU90WlhGR2lDOXM)

### Annotation
#### Evernote
Evernote integration is intended to provide the features and functionality needed for users who use ebook content as part of their self guided leaning or scholarly research.
* [Evernote iOS SDK reference](http://dev.evernote.com/doc/reference/ios/)
* [Evernote API site](http://dev.evernote.com/doc/)

#### Hypothes.is
Hypothes.is an open platform for the collaborative evaluation of knowledge. It will combine sentence-level critique with community peer-review to provide commentary, references, and insight on top of news, blogs, scientific articles, books, terms of service, ballot initiatives, legislation and regulations, software code and potentially eBooks. Efforts are based on the Annotator project and annotation standards for digital documents being developed by the Open Annotation Collaboration for the Web community.
[Hypothes.is](https://github.com/hypothesis)

### Recommendation
#### Goodreads
Let users who are member of Goodreads connect to their Goodreads accounts, and access the books in their shelves, their ratings, their reviews, and their friends – the social reading graph.
* [Good Reads API](https://www.goodreads.com/api)

#### LibraryThing
[LibraryThing API](https://www.librarything.com/services/)