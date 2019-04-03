Many content sources used by libraries cannot be integrated into the Library Simplified ecosystem because they don't have an API that meets our requirements. This page is our attempt to keep track of the content sources used by Simplified partner libraries and explain why they can't be put into the system.

Any content source that wants to integrate with a library needs to answer these three questions:

1. How does the library know what's in the catalog? (Most common answer: MARC records. Runner-up: you don't, just hand over your patron and we'll show them.)
2. How does the content source know that the user is one of our patrons? (Most common answer, at least for NYPL: authentication proxy.)
3. Once the patron makes their choice, how is the content delivered? (Most common answer: web view)

That third one is the deal-breaker for us. A mobile application needs self-contained packages of content.

## Biblioboard

Working on an ODL-based interface, but it's not ready yet.

## Gale Virtual Reference

Ebooks, delivered through a web view. [Available through NYPL](http://go.galegroup.com.i.ezproxy.nypl.org/ps/start.do?p=GVRL&u=nypl&authCount=1), where access is gated and content served through EZProxy.

According to [this page](http://support.gale.com/technical/697), MARC records are made available for the entire collection, but the URL that supposedly gives us the records yields an Internal Server Error.

## Grey House

Publisher. I can't find their library-specific gateway.

## Capstone

Seems to offer [multimedia ebooks](http://mycapstonelibrary.com/login/index.html), most likely delivered through a web page and inserted into a catalog via MARC.

## CAIRN

[Journals in French](http://www.cairn.info.i.ezproxy.nypl.org/), delivered through a web page but available in PDF form on a per-article basis.

## Casalini

[Journals and monographs in Italian](https://www.nypl.org/collections/articles-databases/editoria-italiana-online-fabrizio-serra-journals), delivered through a web interface. PDFs available for some titles but opening them reveals a multi-lingual message saying that your PDF reader is not "recognized". This may be some kind of para-DRM system or it may be actual DRM where a PDF contains its own ACSM file along with a fallback message.

## EBSCOhost

Content is streamed in HTML form through a website that offers note-taking and navigation features. An option exists to "Download this book (offline)". This requires creating a personal EBSCOhost account, independent from your library account. Once you do this, you can check out the book and get an ACSM file.

[The EBSCOhost API](https://support.ebsco.com/eit/ws.php) is a search API. Its search results offer a "pdfLink" but I don't see how to square a link to a full-text PDF to the "create an EBSCOhost account, check out a book, download an ACSM file" process I see on the web site. The most likely explanation is that the full-text PDF link is for open-access works only. That means there is currently no path from a Library Simplified client to an ACSM file.

In summary, all the ingredients are here for a successful integration, but they are not put together in the way we would need.

## Marshall Cavendish

Ebooks, delivered through a web view. [Here's the closest I could get to actual content](http://wdn.ipublishcentral.net/marshall_cavendish/viewinside/46771198771670), and to [the catalog](http://www.marshallcavendishebooks.com/product/adventures-benny).

Not sure how authentication works.

## Salem Press

Ebooks seem to be the focus. [The login site](http://online.salempress.com/) seems to imply the existence of an ILS integration. I don't know what the catalog looks like.

## IndieFlix

Videos. No custom app.

## Zinio

Magazines. Custom app, no API. According to Recorded Books, this content will eventually be available through the One Click Digital API.

## Comics Plus

Comics. Custom app, no API.

## BookFlix/Science Flix/TrueFlix

All three of these sites are from Scholastic and combine ebooks with videos through a web view.

BookFlix is [available at NYPL](http://bkflix.grolier.com.i.ezproxy.nypl.org/sw/node-33982/bk0030pr), where authentication is handled and content is delivered through EZProxy.

## hoopla

[hoopla](https://www.hoopladigital.com/) offers several media formats: ebooks, movies, music etc. They stream content to the patron's device via a custom app.

Patrons choose a library that offers hoopla service, then verify their membership in that library by providing barcode and (optional) PIN.

Hoopla offers an integration API that gives access to catalog and patron information but does not seem to offer an endpoint for delivery of the actual content. Hoopla also uses a wide variety of DRM systems to encrypt and obfuscate different types of content on different platforms.

## Interactive educational resources

Even if we can get the content, the experience these services provide probably won't fit into the Simplified app because the content they deliver is not in a container that we can reasonably render. In most cases the best we can possibly do is open a web view and transparently handle authentication.

### Lynda.com

Lynda.com offers [an API](https://www.lynda.com/integration) that includes authorization hooks and collection information delivered through MARC records. So we could probably show lists of courses in OPDS, and send the patron to a web view once they chose a course.

### Transparent Languages

Custom app, no apparent API.

### Learning Express

Available at NYPL, where authentication happens through and content is served through EZProxy. No visible or advertised API.

### Mango Languages

Available at NYPL, where authentication happens through and content is served through EZProxy. No visible or advertised API.

### Universal Class

No visible or advertised API. [Sales page](http://company.universalclass.com/for-libraries.htm) has few details.