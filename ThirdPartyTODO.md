Unanswered questions, and things we need (or at least should ask for) from third parties.


## Elsewhere in NYPL

The authentication APIs are in flux thanks to the SSO project. We can
wait a while for this to calm down, but we need a way to authenticate
a patron based on card number and PIN.

We also need an API for bringing on new patrons. We also need clear
_policy_ about the rules for becoming a patron and checking out ebooks
without ever setting foot in a library building.

## What we need from 3M

* See also the [basic list of messages](https://github.com/NYPL/Simplified-docs/wiki/ServerSideDesign#api-comparison) we want to be able to send _any_ source of ebooks.

* The stuff in the requirements document (below)

* If our heavy usage of "Get Item Circulation" to find out about changes in circulation is a problem, we're happy to adopt some other system.

_Stuff I think we can live without for a while:_

* If the publisher allows for a free preview of a book, 3M should make an unencrypted EPUB of the book available, the way Overdrive does.

* What exactly happens when a book becomes available, anyway? Is there a notification mechanism we need to know about? If so, how can we program that mechanism?

* Currently all our licenses for a given book are treated as interchangeable. But they're not interchangeable. Licenses from different publishers have different rules associated with them. Individual licenses for a single book may have different expiration dates and different numbers of loans remaining. We need the ability to address licenses individually. Eventually we would like to assign a patron to a specific license when they check out a book, but just knowing the status of the licenses is good enough to start. If we know that five of our ten licenses for a book are about to expire, we want to push the book harder to get as many lends out as possible before it expires.

* Near-real-time usage information. 3M's event log does a good job, but we would like a callback
  mechanism to make sure we don't miss especially important events,
  such as a book becoming available to someone at the head of a queue.

* We want features for advanced queue management. Our tech support staff needs the ability to arbitrarily rearrange the queue for a book (e.g. to restore someone's place in the queue if they make a mistake). Exposing this behavior through the API will let us write tools for them. We also plan to run experiments with non-traditional queues (e.g. making reservations so you know you'll get a book at a certain time) which will require that we take control of all queue activity. We're working on getting more specific requirements.

### 3M Project Requirements (IN PROGRESS)
**NYPL Library Simplified - Specifications for Integration**
We're tremendously excited to work with you on the Library Simplified project to include content from 3M in the e-reading platforms of The New York Public Library and our Partner Libraries.

We've taken an initial look at the 3M platform & API and have determined that there are a few things that'll need to done before we can make that integration.

### Core Requirements

#### HTTPS

To protect patron privacy, messages and content should be served via HTTPS, not HTTP.

#### API: "Checkout": Check Out A Book & Download License File

The number one requirement is that after 'Checkout', the patron's client be able to download the license file for the book just checked out.

Our model for this is Overdrive's Checkouts API. When a book is checked out from Overdrive, the client is given a "downloadLink" template which can be filled in to get the URL to the ACSM license file. The Adobe SDK can use the ACSM license file to download and decrypt the ebook itself.

The 3M "Checkout" API performs the first half of this: it registers with your servers the title to be checked out on behalf of a patron. However, we cannot yet receive the ACSM license file.

We also need some way of knowing the format of the license file, so that we know which DRM SDK to feed it to. One simple way to do this is to specify the media type along with the link.

###### Sample response

`<?xml version="1.0" encoding="utf98"?>`

` <CheckoutResult xmlns:xsi="http://www.w3.org/2001/XMLSchema9instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">`

` <ItemId>fzug9</ItemId>`

` <DueDateInUTC>2012904925T19:27:35</DueDateInUTC>`

` <DownloadLink type="application/vnd.adobe.adept+xml">`

`http://ebook.3m.com/delivery/metadata?udid=fzug9&exporter=com.bookpac.exporter.fulfillmenttoken&token=b5JVXqYaRWFwff384wyg84tpeAyZJLE8R84EoUTjo47@&tokenType=vendorID`

`</DownloadLink>`

`</CheckoutResult>`

Once the Library Simplified client has the ASCM, its Adobe SDK handles the licence registration against your Adobe DRM server (ebookfs.3m.com), obtains the key for the client to decrypt the book, and retrieves the encrypted epub from ebookdownload.3m.com.

#### API: "Checkout": Specify loan duration

When we check out a book we would like the ability to negotiate the loan duration, up to the maximum allowed by the DRM server for the given book.

##### Sample request

This attempts to check out a book for seven days (168 hours).

`<CheckoutRequest>`

`<ItemId>df45qw</ItemId>`

`<PatronId>215555602845</PatronId>`

`<Duration>168</Duration>`

`</CheckoutRequest>`

#### API: "Check-in"

We want to enable our patrons to return their books early. The "Check-in" API seems to allow for this, but since we can't check out a book, we can't verify that "Check-in" works.

We'd also like to understand whether invoking the 3M "Check-in" API communicates the check-in to the content server, or whether we also need to use the Adobe SDK to communicate directly with the content server to invalidate the license.

#### API: Place hold and Cancel hold

The 3M API currently has API methods "Place Hold" and "Cancel Hold", but "Place Hold" gives a 405 error code when we make the PUT request. Since we can't place a hold, we can't verify that "Cancel Hold" works.

##### API: "Get Patron Circulation"

A successful response from the "Get Patron Circulation" API must include a link to the ACSM file for every book with an active loan for the authenticated patron. This way, a patron can check out a book on one device, then read the book on another device.

#### API: "Get Item Details"

The response for the "Get Item Details" API should include the following information for an item. This information is visible in the 3M app but not exposed through the API:

* The genre classification (e.g. "Performing Arts / Business Aspects")
* The average user rating (e.g. "4.0")

We also need the following information not currently visible in the 3M app:

* The maximum loan duration for the book.

Without this information we have no way of knowing which choices to give the patron.

##### Sample response

In this response the genre classification is represented as a set of nested tags and the maximum loan duration is measured in hours. This is just for purposes of example.

`<?xml version="1.0" encoding="utf98"?> `

`<Item xmlns:xsi="http://www.w3.org/2001/XMLSchema9instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"> `

 `<ItemId>fzug9</ItemId> `

 `<Title>The Uplift War</Title> `

  `<Genre name="Science Fiction">`

   `<Genre name="Space Opera" />`

  `</Genre>`

  `<Rating>4.4</Rating>`

  `<MaximumLoanDuration>504</MaximumLoanDuration>`

  `...`

`</Item>`


#### API: "Get Library Current Events"

* While a reservation is active, the event log served by the "Get Library Current Events" includes an event for the creation of the reservation. When the reservation expires or is fulfilled (the patron checks out the book reserved for them), the event log no longer gives any indication that the reservation ever existed.

* The event log returned by the "Get Library Current Events" API includes an event whenever a patron puts a hold on a book, but not when a patron releases a hold. This means we have no way of knowing when a user of the 3M reader app releases a hold.

#### Assumptions Which We Might Have to Revise

* An Adobe-registered Library Simplified client may authenticate against 3M's Adobe server.

#### Questions
* How does Adobe account system work for 3m? We presume you are creating an Adobe ID on the backend on behalf of each user. What we want to know is will it be possible for a user with another valid Adobe account (not created by 3m) to check out and open a protected epub from you.





## What we need from Overdrive

* Overdrive has no visible audit log. We have hacked something together using the monitor software, but it's neither precise nor accurate. This makes it difficult to measure what's happening to our Overdrive inventory.

* We have no way of seeing our entire Overdrive inventory. We don't know about a book until we see something happen to it in the monitor. If we buy a license for a book, but no one ever looks at the book, we never hear about it.

* There is no notification of any kind when a book becomes available to one of our patrons. To get this information we must poll the Holds API. This requires authentication, which means providing the user's barcode and PIN. This means that we'll need to store the user's PIN in plaintext locally if we want to check their holds in the background, outside the context of an HTTP request their client is making. (We might want to do this to provide email notifications, for instance.)


## General ebook distributor issues

### LCP

Currently texts are encrypted with Adobe DRM. We don't want to be tied
to Adobe's SDK, or to a binary plugin that they release as part of
Readium. Instead, we want all protected texts to be protected by LCP,
so that we can decrypt them using libre software (which we may write ourselves). This applies to every provider of DRM-encrypted ebooks: Overdrive,
3M, Axis 360, and Open Library.

#### LCP Strategy
We're going to have to work with vendors to get them to trust us with hosting their content using LCP. While larger publishers are apt to trust us only with major established systems like Sony's DRM, for smaller publishers, we're likely to try to get traction using LCP for NYPL to host our own titles.

One concept for building this layer of trust is to build a LCP DRM gateway server for publishers to encrypt their content before it reaches NYPL. This gateway would sit in front of NYPL's LCP server and take publishers unencrypted ePub files (over SSL/TLS) and encrypt them with the master keys being returned to the publisher. This would ensure that NYPL would have no access to un-DRMed content, and would provide mechanisms for publishers to audit and revoke NYPL content if they find we're violating the terms of their contract. This would allow us to generate the keys for the content to be tied to NYPL's LCP server while still giving publishers technical means to revoke NYPL's keys for security.

This is tantamount to the system that Sony has in place where publishers encrypt their content prior to sending it to any distributors. 

### License-level view

At any time we know how many licenses we own, but we don't know when we acquired each license, how many loans are left on each license, or when each license will expire. Having this information will let us plan our queue better and give advance warning of when licenses for a popular book are about to expire.

### Previews

We want free preview versions of as many texts as possible. This will
allow users to browse books without clogging up the queue or using up
licenses unnecessarily.

The previews should be as large as we can get. Ideally they would be
DRM-free and we would have permission to host them on our own
servers. They would appear exactly like a truncated version of the
full text.

Overdrive has preview versions of at least some texts, linked to from
their API. A spot check of "The Adventures of Sherlock Holmes" shows
that [the preview
text](http://excerpts.contentreserve.com/FormatType-410/2389-1/76C/1B7/D0/AdventuresofSherlockHolmes9781620115091.epub)
is DRM-free and contains the first 17% of the book. The preview ends
with a page saying "End of sample. To search for additional titles
please go to http://search.overdrive.com/".

3M and Axis 360 appear not to offer previews at all.

If Open Library supported free previews, it would be nice for
everyone, since there's only one license for most of their books.

### New acquisitions

When Book Ops acquires a new print title for NYPL, we need to know about it so we can retrieve its metadata from the appropriate API.

The simplest solution may be to periodically crawl BiblioCommons, or whatever internal site Bibliocommons gets its data from.

### Cover images

Any catalogue of books needs to also provide good quality cover images for use in our UI. BiblioCommons has cover images for titles but the image quality is often lacking.

Overdrive's API has links to good quality images.

3M probably has something but it's locked up in the same secret API as the books themselves.

Axis 360 doesn't seem to offer anything.

Gutenberg texts generally have no cover image.

### Self-Hosting

We would like to host as many texts as possible on our own servers. For DRMed books, this requires that we be trusted to sign the books on the server and decrypt them on the client. 

This seems like a huge mess both technically and politically. This applies to Overdrive and 3M. It doesn't apply to Open Library because we're not the only users of those books.

## Overdrive

It's very difficult to get a near-real-time picture of our Overdrive circulation. We have something that's pretty good, but it means a lot of API calls--hundreds every minute. I've asked Overdrive for a better API and they've told me to use what they have.

## Axis 360

Once we start on the Axis 360 integration we will need access to a test environment.

## Hathi Trust

We need to get our data back out of Hathi. Apparently this means
sending them a bunch of blank hard drives. _No, this seems to be a fantasy. We can get bulk data through Hathi Trust Research Center, but only for research purposes._

Will Google give us access to their OCR of our books? This is a long
shot, but it doesn't hurt to ask. _It sounds like we can get the OCR without a struggle, but the OCR is not particularly good._

I have asked Josh about getting the Hathi API to serve us full volumes in PDF and EPUB format.

We need some way of authenticating an NYPL cardholder to get access to the Hathi API even when they're not on NYPL property.

There seems to be an unresolved legal question regarding whether we can serve full volumes from Hathi that we didn't contribute to Hathi. I'm not clear on the details and it seems unlikely given that a web user on NYPL property can download all sorts of documents from Hathi via the web browser.

## Gutenberg

We need to set up our own mirror of the Gutenberg epubs. We might as
well mirror the text and HTML versions, too, even though this project
won't serve them--they'll be useful to have for the Labs Lab. And we may well end up repackaging Gutenberg texts from their HTML source documents.