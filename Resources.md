## APIs, SDKS etc..

* _Italicized bullet items_ are unresolved questions that need attention.

### Administrative Interfaces

#### Active Admin
Active Admin is a Ruby on Rails plugin for generating administration style interfaces. It abstracts common business application patterns to make it simple for developers to implement beautiful and elegant interfaces with very little effort.  It is licensed free of charge under very liberal Creative Commons Licensing which allows redistribution.

[Active Admin GitHub Repo](https://github.com/gregbell/active_admin)

### Content

#### ePUB 3

[IDPF - ePub 3 Overview](http://www.idpf.org/epub/30/spec/epub30-overview.html)

#### BISAC, ONIX and MARC 21

ONIX for Books is the international standard for representing and communicating book industry product information in electronic form. ONIX for Books was developed and is maintained by EDItEUR, jointly with Book Industry Communication (UK) and the Book Industry Study Group (U.S.), and has user groups in Australia, Belgium, Canada, Finland, France, Germany, Italy, the Netherlands, Norway, Russia, Spain, Sweden and the Republic of Korea.

Several organizations have developed mappings from ONIX to MARC21, the most widely-used format for data exchange among libraries. The most comprehensive recent work in this area has been undertaken by OCLC, and we are very happy to make available through the EDItEUR website two papers by Carol Jean Godby of OCLC’s research staff describing the work and its implications, together with detailed mapping tables in Excel spreadsheet form.
The OCLC work – which covers both ONIX 2.1 and ONIX 3.0 – is set in the context of a Metadata Services programm under which publishers’ ONIX files are used to enrich MARC records in the OCLC database with added content, and at the same time MARC elements can be fed back to enhance the usefulness of the publisher’s metadata feed.

[ONIX and MARC 21 Controlled vocabularies](http://www.editeur.org/96/ONIX-and-MARC21/)
[Book INndustry Study Group (BISAC) and ONIX](https://www.bisg.org/onix-books)

### DRM

#### Adobe

The current standard for DRM. The only licit way to unlock a document protected by Adobe
DRM is to use Adobe's SDK ($$$). Adobe is trying to contribute a
binary module to Readium that will allow unlocking documents from
Readium, but both the format (binary) and the license (restrictive)
are unacceptable to the Readium Foundation.

* James expects this will eventually be resolved by Adobe releasing a binary module under a nonrestrictive license. _How is this going_?
* [Readium DRM interface documentation](https://github.com/NYPL/Simplified-docs/blob/master/third-party/Readium%20DRM%20interface.pdf?raw=true)

#### LCP

[Lightweight Content
Protection](http://idpf.org/epub-content-protection) is an open standard for DRM. We have [a draft of the spec](https://github.com/NYPL/Simplified-docs/blob/master/third-party/Lightweight%20Content%20Protection%20(LCP)%20Standard%201.0.pdf?raw=true) and it looks pretty easy to implement. The final standard should be released around May 2014.
* [LCP for Lending](https://github.com/NYPL/Simplified-docs/blob/master/third-party/LCPforLending.pdf?raw=true) is a set of extensions to LCP to support library use cases: destroying a license before its expiration date ("returning" a book), and renewing a license ("renewing" a loan).
* [LCP Overview](https://docs.google.com/a/nypl.org/file/d/0B1GPVlu8pS7MRTc4SVMwRVBlaUg4bFlGWGNzWHZvclhJWGdZ/edit)
* [LCP implementation process flow](https://docs.google.com/a/nypl.org/file/d/0B1GPVlu8pS7MQzZyUUFWQ0RZSDQ/edit)
* We know of [a license server](http://162.243.236.226:8989/manage/) that serves encrypted EPUBs and custom licenses for decrypting them.
* [Readium SDK Module Requirements](https://docs.google.com/a/nypl.org/document/d/1M4mLL_BN2hM0C39-hkw0AzPiSMbq_trq5Z38cptA4Fc/edit#heading=h.eod12t2qp8vg)
* [Readium SDK LCP DRM Module](https://docs.google.com/a/nypl.org/file/d/0B1GPVlu8pS7MaHJBankzeGFpckk/edit)
* We may be implementing our own open source license server and client reader.
* James is negotiating with Overdrive and 3M to get them to serve us books protected with
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
* [Baen Free Library](http://www.baenebooks.com/stanza.aspx) [(in Deutsch)](http://stanza.beam-ebooks.de/)
* [TUEBL](http://tuebl.ca/catalog/) (unfortunately full of pirated stuff)
* [ePub Bud](http://www.epubbud.com/feeds/catalog.atom) (childrens' books)
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
* [OmniLit](https://www.omnilit.com/epub-feed.xml) (not working)

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
#### Readium

* [Readium Annotation Library Scope Document](https://docs.google.com/a/nypl.org/document/d/1NojAfl-jYu3n2AXmssEs9HuH2GNT_y-T5FeMEv3USAI/edit#heading=h.yexjj75e13op)
* [Repo](https://github.com/dmitrym0/web-annotations)
* [SDK Architecture](https://docs.google.com/drawings/d/1fvOQrnKSivIZsl-PVc_KC1STBRcopcBm6I3UiP3HwPA/edit)

#### Evernote
Evernote integration is intended to provide the features and functionality needed for users who use ebook content as part of their self guided leaning or scholarly research.
* [Evernote iOS SDK reference](http://dev.evernote.com/doc/reference/ios/)
* [Evernote API site](http://dev.evernote.com/doc/)

#### Hypothes.is
Hypothes.is an open platform for the collaborative evaluation of knowledge. It will combine sentence-level critique with community peer-review to provide commentary, references, and insight on top of news, blogs, scientific articles, books, terms of service, ballot initiatives, legislation and regulations, software code and potentially eBooks. Efforts are based on the Annotator project and annotation standards for digital documents being developed by the Open Annotation Collaboration for the Web community.
[Hypothes.is](https://github.com/hypothesis)
