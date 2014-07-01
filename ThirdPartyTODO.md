Unanswered questions, and things we need (or at least should ask for) from third parties.

## Elsewhere in NYPL

The authentication APIs are in flux thanks to the SSO project. We can
wait a while for this to calm down, but we need a way to authenticate
a patron based on card number and PIN.

We also need an API for bringing on new patrons. We also need clear
_policy_ about the rules for becoming a patron and checking out ebooks
without ever setting foot in a library building.

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

## 3M

* 3M insists that all patrons use their e-reader to download, decrypt, and read books. This is a deal-breaker. Without this feature, all the stuff below is moot and they simply cannot participate in this project.

* 3M's API has bugs that makes me skeptical it has ever been used. The "Place Hold" and "Release Hold" APIs respond to a PUT request with a 405 error and this message:

<code>&lt;string&gt;The requested resource does not support http method 'GET'.&lt;/string&gt;</code>

* It's not clear whether we must register a patron with 3M before we can make API calls that affect their account.

* 3M does not notify us when someone releases a hold on the book. We are informed about reservations, but only during the 7 days the reservation is active.

* 3M does not provide any classification information for their books, or access to user ratings of the books (information they do have).

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