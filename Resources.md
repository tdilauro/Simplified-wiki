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

The Brooklyn Public Library created a program called [My Library NYC](http://mylibrarynyc.org/about), in which you can sign up online
for a unified library card that works with all NYC libraries. What API
are they using to create a NYPL credential?

The [Innovative Millenium API](http://techdocs.iii.com/patronws_patron_data.shtml) (user:pass nypl_s:chapter) includes a "Patron Update Web Service" which can [create a new patron](http://techdocs.iii.com/patronws_api_operations.shtml#createPatron).

#### Patron authentication

The [Innovative Patron
API](http://vendordocs.iii.com/patron/patronapi.shtml) (user:pass
nypl_s:chapter) can validate a patron based on identifier and PIN.

The [Innovative Millenium API](http://techdocs.iii.com/patronws_patron_data.shtml) (user:pass nypl_s:chapter) probably also does
validation, but I'm not sure how.

### Content

#### Project Gutenberg####

About 45,000 free public domain texts.

We plan to set up a mirror of the Gutenberg ePub documents. This gives us a text source over which we have complete technical and legal control.

* _Set up the mirror._
 - Provision a machine and copy over the ePubs.
 - Retrieve the [MARC records](http://gutenberg.readingroo.ms/cache/generated/feeds/) and insert an "Electronic resource" pointer in each. There are a number of sources for MARC records ([this one](http://www.gutenberg.org/feeds/catalog.marc.bz2) is from gutenberg.org but looks identical to the one at readingroo.ms), and [a third-party script](http://ebooks.adelaide.edu.au/meta/pg/) for converting Gutenberg RDF documents to MARC. 
 - Generate an OPDS feed, either from the MARC records or from the underlying RDF data.
 - The MARC records, the OPDS feed, and the mirror should eventually be updated nightly. The Adelaide third-party library provides daily feeds of new and updated MARC records.
 - Come up with some way of identifying when a Gutenberg text is the same as an Overdrive text, and prefer the Gutenberg text.
* _How many books do we check out through Overdrive that we could replace with Gutenberg?_
* _Are there changes we could make to the default epubs that would improve user experience?_ Doing some manual work on the top 100 books would be worth it.

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

### Open Library

Open Library is a project of the Internet Archive which uses physical
books in libraries around the country as proxies for ebooks which can
be checked out by anyone.

Each Open Library patron must have an individual Open Library account. You can check out up to five books at once. 
Titles are encrypted with Adobe DRM. PDF quality is good. OCR is
atrocious, so EPUBs are not that great.

Open Library has a lot of old (non-public-domain) junk, but it also
has a lot of solid midlist fiction and childrens' books. For instance,
I did a spot check and looked at [38 ebooks tagged "Hugo
winner"](https://openlibrary.org/subjects/hugo_award_winner#ebooks=true)
in the library. There were 19 titles with available copies, 13 titles
that had been checked out, and 6 titles only available in DAISY
format.

Open Library lent out about 140k ebooks in March 2014, which is
significantly better than NYPL did.

#### APIs

[Open Library's API](https://openlibrary.org/developers/api) is a
read-only API that provides bibliographic information about a book and
its editions. It links to ebook versions of a book, but doesn't say
whether copies are available or checked out.

[The Open Library Read API](https://openlibrary.org/dev/docs/api/read)
can look up book identifiers and return bibliographic information,
plus a link to the page where you can borrow that book from Open
Library. This API _does_ specify 'status' as 'full access',
'lendable', 'checked out' or restricted'.

Open Library has [minimal OPDS
support](http://raj.blog.archive.org/2011/03/03/open-library-opds/)
which seems to be undocumented.

Open Library has no API functionality for checking out a book or
joining the queue for a book.


#### Summary

It's tempting to see the nice selection available through Open Library
and want to capture some of it for ourselves. However, the nice
selection will not last too long if we start presenting Open Library books to 100ks of patrons. There
is only one available copy of most books.

It's also hypocritical to integrate our reader into Open Library (and
difficult to get Open Library to cooperate with us) since NYPL does
not _contribute_ any books to Open Library.

#### HathiTrust

* We contributed some 10ks of texts to be scanned. Some 1Ms of public documents are available in total.
* From the premises of a library, anyone can download full documents in PDF format. They can also get EPUB format, but only by using the mobile site.
* Cardholders cannot download full documents off premises because our authentication system is not hooked up to Hathi. (Hathi prefers Shibboleth, used by most university libraries.) 
* Works start with a cover page including a legal disclaimer.
* OCR is bad enough to make the PDF format preferable. 
* There are unanswered questions as to what exactly we can do with these 1Ms of documents and even the 10Ks of documents we originally contributed. However, it looks like a web user on NYPL premises can download a PDF of any of those 1Ms of texts. 

##### The API

* [Haithi DTrust Data API](http://www.hathitrust.org/data_api) API documentation in PDF format.
* Full volumes are available for the Espressnet project only, and only in Espresso Book Machine format. (PDF documentation search term: "Volume-type resources".) Everyone else is restricted to page images, which makes the API useless for our purposes.
 - Leonard asked Josh to pass on the request for Espressnet-like access, and for the ability to get documents in PDF and EPUB format. 
* The [Hathi Trust Research Center](http://www.hathitrust.org/htrc) grants access to bulk data, but only for research purposes.

### Internet Archive

Publishes several million public domain titles, scanned in-house. 
Integration is possible [through
ODPS](http://bookserver.archive.org/catalog/). We can mirror the
alphabetical ODPS feed once, and keep it up to date by periodically
grabbing the "Recent Scans" feed. We can show IA results in search
results and if the user selects one of these texts, have them download
directly from IA (while grabbing a copy for ourselves so we can
directly serve the next person who downloads it.)

IA has a wide selecton of texts but quality is pretty
bad. Bibliographical information is sometimes missing. OCR quality is
very poor. EPUB editions are sometimes [drastically
truncated](https://archive.org/details/AtlasZoologie00Paul) relative
to the PDF edition--we would want to use the PDF edition pretty much
everywhere. Many texts are
[random historical
documents](https://archive.org/details/longtermcarepoli00mass), not "books" as generally understood.

IA also makes available full copies of works I'm pretty sure are still
under copyright, e.g. [A Farewell to
Arms](https://archive.org/details/farewelltoarms01hemi). I'm not sure
what their legal justification is. Note that that is a book scanned
and OCRed by the Internet Archive itself, not contributed by a user
(which is a whole different problem). So whitelisting books provided
by IA won't solve this problem, if it is indeed a problem.

Summary: curated views of IA's catalog can be integrated into our
catalog, but not IMO the entire catalog. It's a junkyard.

### Other sources

Not worth detailed investigation given that they don't host any popular books we can't get elsewhere, but let's keep them in mind.

* [Project Gutenberg Self-Publishing](http://self.gutenberg.org/)
* [WikiSource](http://en.wikisource.org/)

### Art

We have access to a huge variety of non-textual works to use in the interstitial spaces of our application. Think of the way the MTA uses art on the subway to make uniform spaces more interesting and relieve the boredom of waiting. We can display some pre-cached artwork while (e.g.) waiting for a download to complete or for a search to run.

Any topic-based discovery algorithm we use for books should also work for matching artwork to books.

#### DPLA

DPLA has [a comprehensive API](http://dp.la/info/developers/codex/) with liberal usage policies. Access to texts is scattershot. Most of the available texts come from Hathi Trust, which we would access separately, and most texts are of highly specialized interest. The artwork is different story. By integrating with DPLA we get a great source for high-quality interstitial art along with machine-readable metadata about the art.

A random sampling of art: [1](http://collections.si.edu/search/results.htm?q=record_ID%3Asaam_1986.65.386&repo=DPLA) [2](http://www.asia.si.edu/collections/singleObject.cfm?ObjectNumber=F1909.245d) [3](http://collections.nmnh.si.edu/search/anth/?irn=8468479) [4](http://www.americanindian.si.edu/searchcollections/item.aspx?irn=271228) All of these are from the Smithsonian, which (like most DPLA contributors) has generous terms of reuse for educational purposes.

The Cooper-Hewitt museum has an API, and the Smithsonian has an internal API called EDAN, but I believe the DPLA API is the only programmatic access to art from across the Smithsonian.

#### Wikimedia Commons

[A sample](http://commons.wikimedia.org/wiki/Category:PD-Art_%28PD-US-not_renewed%29) of [public domain art](http://commons.wikimedia.org/wiki/Category:PD-Art_%28PD-US%29) available through Wikimedia Commons. Access is through the MediaWiki API. Metadata is not as good as for DPLA materials.

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

BiblioCommons also does not support lifecycle events such as checkout. The only thing it knows is the URL to the Overdrive/3M page for a title. We send the user over to the vendor site, and neither we nor BiblioCommons knows what goes on on that site.

Because of this, we believe there is no point in integrating BiblioCommons with the app. Since BiblioCommons is how people will be checking out ebooks when they're not using the app, it may make sense to integrate any new book sources (such as Gutenberg) with BiblioCommons.

The BiblioCommons API is read-only and heavily focused on publicly available catalog data. It doesn't have any special knowledge of ebooks. The status of a given copy of a book (e.g. "checked out") is a user-specific field that we would have to specify ourselves. There is an interesting feature called "lists", giving us access to lists made by users of the web site, but again, it's all read-only.

Leonard created an API key for use by NYPL Labs. It can be found in the shared keyring.

* [BiblioCommons] (http://developer.bibliocommons.com/blog/read/Welcome_to_the_BiblioCommons_API)
* [API documentation](http://developer.bibliocommons.com/docs)
* [Interactive API](http://developer.bibliocommons.com/io-docs)

### OPDS

[OPDS](http://opds-spec.org/specs/opds-catalog-1-1-20110627/) is an
Atom profile for describing catalogues of electronic publications. 

* Uses Atom's vocabulary for describing a book's metadata. Encourages the use of Dublin Core vocabulary except where Atom uses a different term for the same thing.
* Uses Atom feeds to describe lists of publications, which can be filtered or ordered according to facets. The concept of a facet is defined, but no particular facets are defined.
* Defines vocabulary for describing feeds of "new", "recommended", or "popular" items, as well as a user's "shelf" of acquired items.
* Uses OpenSearch as a search protocol.
* Defines a vocabulary for basic transitions (notably "borrow"), but does not define a protocol for carrying out those transitions.

As seen on [a comparison chart on Wikipedia](http://en.wikipedia.org/wiki/OPDS#Comparison_of_OPDS_clients), a number of e-reading applications function as OPDS clients. Several bookstores (see below) offer OPDS feeds that can be presented to users of those applications, who can buy books from inside the application.

I think OPDS is a good place to start for a catalogue implementation. It's widely supported and it reinvents as little as possible--mostly bookstore/library vocabulary.
We might not want to serve Atom documents between the backend and the mobile client, since Atom is an XML format, but it shouldn't be hard to serve a [Siren](https://github.com/kevinswiber/siren) version of an OPDS document. 

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