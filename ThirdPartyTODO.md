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

Currently texts are protected by Adobe DRM. We don't want to be tied
to Adobe's SDK, or to a binary plugin that they release as part of
Readium. Instead, we want all protected texts to be protected by LCP,
so that we can write a free implementation. This applies to Overdrive,
3M (?), and Open Library.

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

Availability of previews from 3M are unknown.

If Open Library supported free previews, it would be nice for
everyone, since there's only one license for most of their books.

### New acquisitions

When Book Ops acquires a new title for NYPL, we need to know about it so we can retrieve its metadata from the appropriate API.

The simplest solution may be to periodically crawl BiblioCommons, or whatever internal site Bibliocommons gets its data from.

### Cover images

Any catalogue of books needs to also provide good quality cover images for use in our UI. BiblioCommons has cover images for titles but the image quality is often lacking.

Overdrive's API has links to good quality images.

Gutenberg texts generally have no cover image.

### Self-Hosting

We would like to host as many texts as possible on our own servers. For DRMed books, this requires that we be trusted to sign the books on the server and decrypt them on the client. 

This seems like a huge mess both technically and politically. This applies to Overdrive and 3M. It doesn't apply to Open Library because we're not the only users of those books.

## Overdrive

Many planned features require that we be able to measure the velocity
of a license (i.e. how frequently it changes hands). To do this we
need an accurate, near-real-time picture of our Overdrive
inventory. We don't have anything close.

Currently we know which titles we own, but not how many copies of
those titles we've licensed, how many lends remain on those copies,
how many copies of a title are currently checked out, or how many
people are in the queue for a title.

When a patron checks out a book, we don't hear about it. They
currently can't return books, but if they could, we wouldn't hear
about that either.

We can get inventory information for a single book via the Overdrive
API, but since we don't get notified when the status of a book
changes, we have no idea which books need to be updated.

If we could instantly switch everyone over to using our e-reader, we
could solve this problem, since we can have the server-side API log
inventory activity as it happens. But for the forseeable future there
will be people checking out ebooks on the Overdrive and 3M websites.

A reporting API is (probably) not good enough. Possible solutions
include a feed of events that affect our inventory, or (in the short
term) regular dumps of our inventory that are no older than N minutes.

## 3M

Nothing is known about the 3M API. In particular, we don't know if it
has the same inventory visibility problems as Overdrive. But it
probably does. The first step is to get basic access and information about their API.

## Hathi Trust

We need to get our data back out of Hathi. Apparently this means
sending them a bunch of blank hard drives.

Will Google give us access to their OCR of our books? This is a long
shot, but it doesn't hurt to ask.

If Hathi will only serve us books that we contributed for scanning,
then after we get our data back out we may not need their API at
all. We can serve the PDFs ourselves.

To the extent that we do need the API: Hathi's API can serve full
volumes, but it will only serve them to the EspressNet project, in
Espresso Book Machine format (a modified version of PDF). We need to
be on the allow list, and we need to be able to get books in PDF
format.

## Gutenberg

We need to set up our own mirror of the Gutenberg epubs. We might as
well mirror the text and HTML versions, too, even though this project
won't serve them--they'll be useful to have for the Labs Lab. And we may well end up repackaging Gutenberg texts from their HTML source documents.