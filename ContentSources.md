### Self Published Fan Fiction
WattPad - 
Person - Heather McCormack
Medium (OpEd, Publication "matter")

Bi-liner
Long Reads - 
Academic - 
kuali Ole - 
Contra Costa - 
FeedBooks - 

###Self Publish 
####Smashwords - 
See What is Free and Popular -  uncurated

###Academic Market - Multi Use

L'Sevier
Springer
Wiley
O'Reily Media
University Press

####University Press (Class of Publisher) 
#####Platforms - Rebeca Federman
Project Muse
jStor
Oxford Scholarship Online (OPSO) - Unlimited Multi-use

###Street Literature
Kensington - curated collection

## DRM-encrypted books

3M, Overdrive, and Axis 360 are the main vendors of DRM-encrypted ebooks. Each has a custom API that covers more or less the same ground. [Server-Side Design](https://github.com/NYPL/iOS-Reader/wiki/ServerSideDesign) has a back-to-back comparison of 3M, Overdrive, and Axis 360 when it comes to the integration features we care about. I won't go into the details here.

### Overdrive

* [OverDrive APIs] (https://developer.overdrive.com/apis) are very well-designed in general.
* The main sticking point is getting an ongoing picture of inventory.
* [API for checking availability](https://developer.overdrive.com/apis/library-availability)
* [API for checking out a book and "locking in" a format (e.g. "ebook-epub-adobe") to get a download URL.](https://developer.overdrive.com/apis/checkouts)
* [API for downloading a book in the locked-in format.](http://developer.overdrive.com/apis/download)
 - _What format is the downloaded file? ODM? ASCM? We need to be able to open it in our reader._
* No API for returning a book early. ("If a format has been locked in, then users need to use software like OverDrive Media Console or Adobe Digital Editions to complete the return process.")
 - _We need to be able to do this from our reader._
* Overdrive's web site has a separate section offering about 27000 ebooks from Project Gutenberg. The presentation is awful. This is probably the ~29000 ebooks from the 2010 PG DVD.

### 3M Cloud Library

* We have a copy of the secret documentation.
* The primary missing functionality here is that there is no way to download a DRM-encrypted book. 3M wants everyone to use their reader.

### Axis 360

* We have a copy of the secret documentation.
* Seems to have every major feature we need.

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

## Open-access books

### Project Gutenberg

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

### unglue.it

Only about 20 books currently, but they're high quality open access stuff. There is an unknown integration API that requires a library partner account. _James is investigating this._

### HathiTrust

* We contributed some 10ks of texts to be scanned. Some 1Ms of public documents are available in total.
* From the premises of a library, anyone can download full documents in PDF format. They can also get EPUB format, but only by using the mobile site.
* Cardholders cannot download full documents off premises because our authentication system is not hooked up to Hathi. (Hathi prefers Shibboleth, used by most university libraries.) 
* Works start with a cover page including a legal disclaimer.
* OCR is bad enough to make the PDF format preferable. 
* There are unanswered questions as to what exactly we can do with these 1Ms of documents and even the 10Ks of documents we originally contributed. However, it looks like a web user on NYPL premises can download a PDF of any of those 1Ms of texts. 

#### The API

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

IA has a wide selection of texts but quality is pretty
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

## Art

We have access to a huge variety of non-textual works to use in the interstitial spaces of our application. Think of the way the MTA uses art on the subway to make uniform spaces more interesting and relieve the boredom of waiting. We can display some pre-cached artwork while (e.g.) waiting for a download to complete or for a search to run. This is an easy way to give even a generic e-reader some personality and a "library" feel.

Any topic-based discovery algorithm we use for books should also work for matching artwork to books.


###Sendetics
### ChileFresh.com
### CoverArt

### DPLA

DPLA has [a comprehensive API](http://dp.la/info/developers/codex/) with liberal usage policies. Access to texts is scattershot. Most of the available texts come from Hathi Trust, which we would access separately, and most texts are of highly specialized interest. The artwork is different story. By integrating with DPLA we get a great source for high-quality interstitial art along with machine-readable metadata about the art.

A random sampling of art: [1](http://collections.si.edu/search/results.htm?q=record_ID%3Asaam_1986.65.386&repo=DPLA) [2](http://www.asia.si.edu/collections/singleObject.cfm?ObjectNumber=F1909.245d) [3](http://collections.nmnh.si.edu/search/anth/?irn=8468479) [4](http://www.americanindian.si.edu/searchcollections/item.aspx?irn=271228) All of these are from the Smithsonian, which (like most DPLA contributors) has generous terms of reuse for educational purposes.

The Cooper-Hewitt museum has an API, and the Smithsonian has an internal API called EDAN, but I believe the DPLA API is the only programmatic access to art from across the Smithsonian.

### Europeana

Basically the DPLA for European institutions. I have not investigated [the API](http://pro.europeana.eu/web/guest/api) because there's no point until we decide whether to do anything with DPLA data, but it looks pretty similar to DPLA. [A sample of the available data.](http://labs.europeana.eu/data/)

### Wikimedia Commons

[A sample](http://commons.wikimedia.org/wiki/Category:PD-Art_%28PD-US-not_renewed%29) of [public domain art](http://commons.wikimedia.org/wiki/Category:PD-Art_%28PD-US%29) available through Wikimedia Commons. Access is through the MediaWiki API. Metadata (whether relating to source or to topics) is not as good as for DPLA materials.

### Instagram

Joe did a Shipit Day project involving pulling Instagram projects tagged #NYPL and using them as background images.
