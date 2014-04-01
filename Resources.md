## APIs, SDKS etc..

#### Authentication
[OAuth2.0 Client for iOS](https://github.com/AFNetworking/AFOAuth2Client) MIT License

[Patron API](http://vendordocs.iii.com/patron/patronapi.shtml)
[Millenium API](http://techdocs.iii.com/millennium-api.shtml)
user/pass:nypl_s/chapter

###Catalogue Data APIs###
#### BiblioCommons####
BiblioCommons is an innovative social discovery layer for public libraries, that combines powerful search and account management capabilities with a social sharing and discovery platform

* [BiblioCommons] (http://developer.bibliocommons.com/blog/read/Welcome_to_the_BiblioCommons_API)

### Content

#### Gutenberg####

About 45,000 free texts.

* _Let's set up our own mirror of the ePub documents._
* _How many books do we check out through Overdrive that we could replace with Gutenberg?_
* _Are there changes we could make to the default epubs that would improve user experience?_ Doing some manual work on the top 100 books would be worth it.

#### Internet Archive / Open Library ####

#### Overdrive

* [OverDrive APIs] (https://developer.overdrive.com/apis)
* [API for checking availability](https://developer.overdrive.com/apis/library-availability)
* [API for checking out a book and "locking in" a format (e.g. "ebook-epub-adobe") to get a download URL.](https://developer.overdrive.com/apis/checkouts)
* [API for downloading a book in the locked-in format.](http://developer.overdrive.com/apis/download)
* _What format is the downloaded file? ODM? ASCM? We need to be able to open it in our reader._
* No API for returning a book early. ("If a format has been locked in, then users need to use software like OverDrive Media Console or Adobe Digital Editions to complete the return process.") _We need to be able to do this from our reader._

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

### Reading
#### Repositories from Readium Foundation
Readium SDK is an open source project developing a productized, high-performance, cross-platform rendering engine for EPUB 3 content, optimized for use in native applications (mobile/tablet and secondarily desktop systems). Simplistic test applications for Android, iOS, OS/X and Windows are part of the SDK, along with test frameworks. Readium SDK is extensible in various areas including DRM support. Readium SDK will be dual-licensed under both GPL and a commercial-friendly (non-copyleft) license. Note that the default license currently in the open source code on github is GPL.

A separate sub-project intends to in parallel develop an interoperable implementation of a lightweight content protection technology (DRM) that can be implemented into the modular framework of Readium SDK. This DRM will be available under a separate license, terms and conditions TBD.
* [Readium SDK](http://readium.github.io/sdk-api-doc/)
* [Readium Documents on Google Docs] (https://drive.google.com/a/nypl.org/?hl=en&pli=1#folders/0BzaNaBNAB6FjbU90WlhGR2lDOXM)