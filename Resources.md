## APIs, SDKS etc..

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

The [http://techdocs.iii.com/patronws_patron_data.shtml](Innovative
Millenium API) (user:pass nypl_s:chapter) probably also does
validation, but I'm not sure how.

###Catalogue Data APIs###
#### BiblioCommons####
BiblioCommons is an innovative social discovery layer for public libraries, that combines powerful search and account management capabilities with a social sharing and discovery platform

* [BiblioCommons] (http://developer.bibliocommons.com/blog/read/Welcome_to_the_BiblioCommons_API)

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
Haithitrust Data API is a programatic access layer to their catalogue repository.
* [Haithi DTrust Data API](http://www.hathitrust.org/data_api) Documentation in PDF format.
* Full volumes are available for the Espressnet project only, and only in Espresso Book Machine format. (Search term: "Volume-type resources".) Everyone else is restricted to page images.
 - _We need access to full documents in ePub or PDF format._ 
* _How much access can we get? Do we have access to images of in-copyright books?_
* _How do we get Hathi Trust books into Bibliocommons so that they show up in our catalogue?_
* Google owns the OCR data and Hathi has a copy but cannot make it available to us.
 - _Is this true? Can we get access to the data by making a deal with Google?_
 - _Can we do the OCR ourselves on a proactive or on-demand basis?_
 - _Even if we can't have direct access to the OCR, can we do full-text search?_

### DRM

Currently the only licit way to unlock a document protected by Adobe
DRM is to use Adobe's SDK ($$$). Adobe is trying to contribute a
binary module to Readium that will allow unlocking documents from
Readium, but both the format (binary) and the license (restrictive)
are unacceptable to the Readium Foundation. James expects this will
eventually be resolved, but it may lock us into using Readium when we
don't want to.

* [Readium DRM interface documentation](https://github.com/NYPL/iOS-Reader/blob/master/Documents/Readium%20DRM%20interface.pdf?raw=true)

In May 2014, the first implementation (? final standard?) of [Lightweight Content
Protection](http://idpf.org/epub-content-protection) will be
released. (By who?) This will be significantly easier to use and less
onerous than Adobe DRM. James is trying to get us early access to LCP
so we can write our own implementation. James will also be negotiating
with Overdrive and 3M to get them to serve us books protected with
LCP.

* [Draft of LCP standard](https://github.com/NYPL/iOS-Reader/blob/master/Documents/Lightweight%20Content%20Protection%20(LCP)%20Standard%201.0.pdf?raw=true)

### Lifecycle

Each content provider has its own workflow for checking out books. Because of this, we have a very limited view into which books have been checked out and what books are currently available. The reader will send all lifecycle events through an API as yet to be defined, which will dispatch to the appropriate content provider.

If nothing else, this will let us keep track of what books are being checked out. Many advanced lifecycle features depend on this--for instance, queuing requires that we know when a given book will be available.
The simplest way to track the lifecycle of a loan may be to integrate with NYPL's existing loan infrastructure.

Maintaining a detailed knowledge of what books have been checked out will also require changing the website to send loan lifecycle events to an NYPL server before/after sending the user to the third-party website where the book actually gets checked out. This will be very unreliable.

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