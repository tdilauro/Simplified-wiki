# OneClick Integration
This document is to help you connect your library's OneClick collection to your Library Simplified circulation manager.


## Getting Access:
OneClick provides trusted partner tokens.  You'll need one to talk to their api.  To obtain your tokens, you'll need to contact OneClick via email or Slack.


## API Basics:
OneClick provides [API Documentation](http://developer.oneclickdigital.us/documents/getting-started). 

The API attempts to be as RESTful as possible.  Most request parameters are passed in the URL, and the http method the request is sent by helps differentiate between related request types.  For example, let's say you have a library whose OneClick id is 123.  In that library's OneClick collection, you have a book with the ISBN number 456.  And you have a patron with OneClick id 789.  Then, to ask to check out this book to this patron, you would sent a POST request to the URL [api_url]/123/patrons/789/checkouts/456.  And to ask to return the book, you would send a request to the same URL, but through a DELETE http method.

Most OneClick responses are in the JSON format.  You'll either get a data structure containing the information you asked for, or a dictionary with an error message.  Here, OneClick puts HTTP response codes to intensive use.  Bad requests, data conflicts, books not found are all indicated through HTTP 500, 404, 409, etc. codes.  The error message then provides a more detailed explanation of the problem.


## Patron Account Management:
We store patron account data in our ILSes, and pass it through to OneCLick.  Thus, our ILS is the One True Source to keep track of.


## Start Your Catalog:
To know what titles to show your customers, you'll need to populate your database with your library's OneClick catalog.  You can get this by running the ___ script.  The script sends a GET request to [api_url]/[library_id]/media/all .  This request is quite taxing on OneClick servers, so use sparingly and don't DOS them :).

OneClick currently has collections of ebooks, eaudio, and emagazines.  Library Simplified currently handles ebooks, with eaudio in the pipeline.  The magazine collections are handled through a separate API on the OneClick side, and are to become more integrated in the future.  They are further removed in the Library Simplified's pipeline.


## Maintain Your Catalog:
Once the catalog is loaded into the database, you'll need a way to check for and import any changes.  Do this with the ___ script.  The script will call the [api_url]/[library_id]/media/delta?begin=YYYY-MM-DD&end=YYYY-MM-DD endpoint, described in the API [here](http://developer.oneclickdigital.us/endpoints/titles#get-calculated-deltas).


## Book Availability:
There is no way to tell how many licensed copies of a book are available for lending.  All we can know, is the binary answer to whether the book was available when the last catalog availability check was run.  If the book is marked "available", then the patron may try to check it out.  If the book's licenses have gotten used up in the time between our availability check and the checkout request, an appropriate error message will be displayed, the patron will have the opportunity to put the book on hold, and the availability flag in our database will be updated.


## Search The Catalog:
The API call is described [here](http://developer.oneclickdigital.us/endpoints/search-partner).  Your search capabilities are not completely flexible -- there is a set list of facet and filter values to search by.  For more flexibility, you'll be using the Library Simplified search functionality, rather than the direct call to OneClick API.


## Checkouts:
Resources are identified by library id, patron id, and item id.  The library and patron ids are OneClick-internal.  The item id is usually an ISBN.  See [here](http://developer.oneclickdigital.us/endpoints/title-checkouts-admin) for the list of available functionality.  

If the book is not available, or if the patron is not allowed to check out (for example, if they've reached their book quota), you'll get an HTTP error status and a response text with a more detailed error message.  

An HTTP POST call performs a checkout.  A DELETE call returns a book.  And a GET call returns the patron's current checkouts.

The book is checked out for X number of days, where X is determined by your library's agreement with OneClick.  You can shorten the amount of time a book is checked out for by passing the optional "days" parameter.  You cannot extend the checkout time range with the "days" parameter past the library's default X.  You will be able to extend the load by sending a renew call.


## Holds:
Work like checkouts, but with books in the catalog that are not currently available.  If you attempt to put a hold on a book that is currently available, your hold request will succeed, and then a bit later, a OneClick daemon will silently convert the hold record to a checkout record.


## Wishlists:
These exist to help libraries know which books their patron would like them to acquire.  Library Simplified does not currently implement this request functionality.


## Bookmarks:
Currently the functionality is limited to audiobook playback positions.
Assume an audio book is available in one format/encoding quality.







