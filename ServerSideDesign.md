## The shape of the problem

*We deliver texts to readers from sources.*

There are many sources of texts:

* Public domain sources like Project Gutenberg, Internet Archive, and
  Hathi Trust.
* Paid sources like Overdrive, 3M, and Axis 360.
* The electronic catalogs of other libraries. (Open Library is an
  example of this, albeit an odd example.)
* Library branch buildings.

These sources give us texts under more or less restrictive terms. They
also give us bibliographic records--_information_ about texts--under
very liberal terms.

For some of these sources, we can keep a local copy of both the texts
and the records (Gutenberg). For some we can keep a local copy of the
records, but the texts are way too large to mirror (Internet Archive),
or we may not have permission to mirror the texts (Overdrive). For
some sources (other libraries' catalogues) it may not be possible to
even keep a local copy of the records.

Some sources allow free access to texts. Some will only serve a
DRM-encrypted text to a limited number of simultaneous readers, a
limited number of times.

Some sources serve nice-looking EPUB documents. Some serve only
PDFs. Some can serve either EPUB or PDF, but the EPUB always looks
terrible and is full of OCR errors. Some sources only serve physical
books.

_Users want access to books._ Of all the distinctions covered here,
the only one that concerns them enough to affect the interface is
physical vs. electronic. Users will also care about distinctions not
covered here: the language a book is written in, and whether or not
it's an audiobook.

Although we will be focusing on iOS devices initially, readers will
expect to use all sorts of computers to get their books: iPhones,
iPads, Android phones and tablets, Kindles, Nooks, Sony readers,
Kobos, and ordinary computers.

## The basic architecture

The server implementation does the work of a library. It makes
unprompted and prompted book recommendations. It can look up a
bibliographic record. Through its interface the reader can check out
or return a book. If a book is not available through the interface,
the reader can put a hold on it and get it later--whether "get it
later" means waiting a few months for an electronic version or walking
to a branch library for a print version.

The server provides a uniform view to all sources of texts. Not all
texts support all operations. An open-access source has no need to
support "hold" or "return a book" because its inventory is not
scarce.

If the server cannot mirror a source's metadata and keep it up to
date, that source cannot be a full participant in book
recommendations. "Metadata" here _especially_ includes the currently
available inventory. This means you, OverDrive. (It also means Open
Library, and possibly Axis 360.) The best way for the server to keep
its metadata in sync with the source is to listen to the source for
notifications or to periodically fetch an event feed.

## The Netflix Strategy 

Some e-reading applications feature an enormous selection of books you
can buy from an online bookstore. Public library buildings have a
mediocre selection of books you can read for free. Our e-reading
application has a truly awful selection of books you _might_ be able
to read for free.

Copying the "storefront" interfaces of other e-reading applications
(as BiblioCommons does) will lead us to promise more than we can
deliver. I think it's a good idea to copy the feel of a physical
library (shelves with in-stock books on them and no markers for
checked-out books), but I'd also like to introduce concepts from
another source: Netflix.

Netflix's streaming service lets you watch videos for free, after
paying a monthly fee. They have a problem: their selection is
_terrible_. If you search for what you want to watch on Netflix, the
answer is usually crushing disappointment or "how about ordering this
on DVD?". And yet the Netflix streaming video service is very
successful. Why?

1. Their recommendation engine is really good at finding the subset of
   their awful selection that _you_ are interested in watching. ("_The
   Conversation_, 1970s Gene Hackman thriller, never heard of it, but
   sure, looks good.")
2. If something interests you, instead of committing to watch it
   right now, you can put it on a list and come back to it when you have
   time.
3. They publish exclusive content that you can't get anywhere else.

3 is not really relevant to us (or is it?...), but I propose we learn
from 1 and 2.

In one respect, our selection is worse than Netflix. While they have
the rights to a title, an unlimited number of people can stream that
title simultaneously. We can only offer a title to around 1-10 people
simultaneously. The effective inventory we can offer a given patron is
much smaller than our actual inventory, so finding the gems is even
more important.

In one respect, our selection is better than Netflix. We have free
access to the public domain. There are public domain books that are
still very popular, and more appealing to readers than public domain
movies are to Netflix users. But the vast majority of the public
domain is awful, lacking even the appeal of novelty. Making sense of
this cornucopia requires some way of finding the gems in the junk.

Conclusion: focus on feeds of recommendations. If someone does a
search, the answer is probably going to be "how about checking that
out of a branch library?" or "how about standing in line for 3
months?" So downplay searches as much as possible--just like physical
libraries do.

## Things the client might tell the server

A combination of OPDS, LCP, and plain HTTP will cover most of this.

* Give me a feed of recommendations (OPDS)
* Give me a feed of search results (OPDS+OpenSearch)
* Give me a preview of this book (HTTP)
* Give me this book (possibly with license) (HTTP or LCP)
* I want to extend my license for this book (LCP)
* I want to surrender my license for this book (LCP)
* Put this book on hold/put me in the hold queue for this book (undefined)
* Release my hold on this book/remove me from the hold queue for this book (undefined)

## Things the server might tell a source

* Tell me about your entire inventory.
* Tell me everything that happened to your inventory recently.
* Give me a feed of search results.
* Give me a preview of this book.
* Give me this book licensed to this patron.
* Extend this patron's license for this book.
* This patron is surrendering their license for this book.
* Put this book on hold for this patron/put them in the hold queue.
* Release this patron's hold on this book/remove them from the hold queue.

## Things a source might tell the server

* Something happened to my inventory (e.g. someone checked out a book,
  a book was added to inventory).

## API comparison

<table>
 <tr>
 <th>Message from server to source</th>
 <th>Overdrive</th>
 <th>3M</th>
 <th>Axis360</th>
 </tr>

<tr>
<td>Tell me about your entire inventory</td>
<td><a href="https://developer.overdrive.com/docs/products-link">via Products link?</a> </td>
<td>"Get Library Events" + "Get Library Purchase Count"</td>
<td>"Availability" with "updatedDate" to get historical information, then "Availability" to get information about specific books</td>
</tr>

<tr>
<td>Tell me what happened to your inventory recently</td>
<td><a href="https://developer.overdrive.com/apis/search">Search</a> with lastupdatetime + follow-up with <a href="https://developer.overdrive.com/apis/library-availability">Library Availability</a>?</td>
<td>"Get Library Events"</td>
<td>"Availability" with "updatedDate"</td>
</tr>

<tr>
<td>Give me a feed of search results</td>
<td>Search API</td>
<td>Not supported</td>
<td>Not supported</td>
</tr>

<tr>
<td>Give me a preview of this book</td>
<td>Linked to from book detail</td>
<td>*Not supported*</td>
<td>*Not supported*</td>
</tr>

<tr>
<td>Give me this book licensed to this patron</td>
<td>Checkouts API + Download API</td>
<td>"Check Out", but *no way of getting the actual book*</td>
<td>"DRM Create"/"DRM Setup" + "Checkout"
</tr>

<tr>
<td>Extend this patron's license for this book</td>
<td>Not supported</td>
<td>Not supported</td>
<td>Not supported</td>
</tr>

<tr>
<td>This patron is surrendering their license for this book.</td>
<td>Not supported--this is a DRM operation</td>
<td>"Check In" (but DRM operation also required)</td>
<td>Not supported (probably a DRM operation)</td>
</tr>

<tr>
<td>Put this book on hold for this patron/put them in the hold queue.</td>
<td>Holds API/"Place a hold"</td>
<td>"Place Hold"</td>
<td>"Add to hold"</td>
</tr>

<tr>
<td>Release this patron's hold on this book/remove them from the hold queue.
</td>
<td>Holds API/"Removing a hold"</td>
<td>"Cancel Hold"</td>
<td>"Remove Hold"</td>
</tr>

<tr>
<td>View all of this patron's holds and checkouts.</td>
<td>Linked to from patron representation</td>
<td>"Get Patron Circulation"</td>
<td>*Not supported*</td>
</tr>

</table>

## What we need from 3M

* The number one requirement is that after 'Checkout', I be able to
  download the license file and encrypted ebook file for the book I
  just checked out. See, for example, [Overdrive's Checkouts API](https://developer.overdrive.com/apis/checkouts). When I check out a book from Overdrive I am given a ["downloadLink"
  template](https://developer.overdrive.com/apis/download) which gives me the URL to the ACSM
  license file, which contains the URL to the actual ebook.

* 3M does not serve free previews of books, even if free previews have been negotiated with the publisher.

* Currently all our licenses for a given book are treated as interchangeable. But they're not interchangeable. Licenses from different publishers have different rules associated with them. Individual licenses for a single book may have different expiration dates and different numbers of loans remaining. We need the ability to address licenses individually. Eventually we would like to assign a patron to a specific license when they check out a book, but just knowing the status of the licenses is good enough to start. If we know that five of our ten licenses for a book are about to expire, we want to push the book harder to get as many lends out as possible before it expires.

* 3M tracks user ratings for books, but doesn't make that information available in "Get Item Details". We'd like access to that information. If there's any more detailed bibliographic information about a book
that we don't currently get, it would be nice to get it, but the big
one is the rating.

* Near-real-time usage information. 3M's event log does a good job, but we would like a callback
  mechanism to make sure we don't miss especially important events,
  such as a book becoming available to someone at the head of a queue.

* What exactly happens when a book becomes available, anyway? Is there a notification mechanism we need to know about? If so, how can we program that mechanism?

* Advanced queue management. Our tech support staff needs the ability to arbitrarily rearrange the queue for a book (e.g. to restore someone's place in the queue if they make a mistake). Exposing this behavior through the API will let us write tools for them. We also plan to run experiments with non-traditional queues (e.g. making reservations so you know you'll get a book at a certain time) which will require that we take control of all queue activity. 

## What we need from Overdrive

* Overdrive has no visible audit log. We have hacked something together using the monitor software, but it's neither precise nor accurate. This makes it difficult to measure what's happening to our Overdrive inventory.

* We have no way of seeing our entire Overdrive inventory. We don't know about a book until we see something happen to it in the monitor. If we buy a license for a book, but no one ever looks at the book, we never hear about it.

* There is no notification of any kind when a book becomes available to one of our patrons. To get this information we must poll the Holds API. This requires authentication, which means providing the user's barcode and PIN. This means that we'll need to store the user's PIN in plaintext locally if we want to check their holds in the background, outside the context of an HTTP request their client is making. (We might want to do this to provide email notifications, for instance.)

## July '14 Server Design Elements
For our first test build, targeted for the end of July 2014, we will need a live server performing authentication, search, and delivery of our Project Gutenberg corpus.

This will include:
* User Model
* Authentication
* Reading events to keep on server - Things that they have right now
* App debugging information
* Navigation feed [in whatever language - i.e. books in language] [mostly statically generated]
* Recommended lanes -- not paginated
* Other feeds [OPDS acquisition feeds] will be paginated - next link to next page. Will follow OPDS pagination guidelines
* Sorts [OPDS "facets"] - By title, By Author - facet groups will be sent to client
* Search: [Title + Author] - might not be fast

This work temporarily puts on hold:
* Cover images
* Book Description
* Book Quality Assessment
* Permanence of database
* Hierarchy + quality of hierarchy

Authentication will be handled by connecting to the [iii authentication server](https://github.com/NYPL/Simplified-docs/wiki/Authentication#innovative-patron-api).

Leonard is investigating the viability of Heroku as an architecture for this server, but it might ultimately not make sense to keep it there.