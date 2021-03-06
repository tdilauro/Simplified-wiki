## Overdrive requirements document (sketch)

### DRM credential management

We currently have no visibility into the DRM credentials associated with a patron's account, whether they were created by Overdrive, or created by the patron and manually associated with the patron's Overdrive account. We ask for an API similar to the one offered by Baker & Taylor, with the following four pieces of functionality:

1. Associate an existing DRM credential with the patron's Overdrive account.
2. List the DRM credentials currently associated with the patron's Overdrive account.
3. Create a new DRM credential and associate it with the patron's Overdrive account.
4. Disassociate a DRM credential from the patron's Overdrive account. We do not require that this functionality work for credentials originally created by Overdrive.

Access to this functionality would be controlled by Overdrive's existing patron token.

The API should be support different kinds of DRM credentials for the same patron (e.g. a patron should be able to have an Adobe ID and a URMS ID).

### Circulation tracking

We currently use three techniques for tracking the circulation of our
Overdrive materials:

1. Every minute, we invoke the `search` API to find all the records
   that have changed since since the last minute. We use the library
   availability API to check on the availability for each book
   mentioned.

   We have found that this technique misses a lot of circulation events, so we use the other techniques to fill in the gaps.

2. We use the `products` link to crawl our entire collection, using
   the library availability API to check on availability for each item. A complete
   crawl takes approximately 4 hours, though by parallelizing the
   library availability calls we could speed this up considerably.

3. We use the `products` link to crawl our collection, as in #2, but we only look at the end of the list. 
   This works because recently changed items tend to congregate at the end of the list. This is much faster 
   than #2, and more effective than #1.

Even with all these systems in place there are events we don't see. For example, when a book is checked in and then immediately checked out to the next patron in the hold queue, the only thing we see is the number
of holds decreasing by one. We don't see that the book was checked in or that it was checked out again. It looks like someone gave up on waiting for the book.

A persistent event log would be a solution that worked for us. The
link to this log might look like this:

`http://api.overdrive.com/v1/collections/[collection]/circulation`

The representation of that resource might look like this:

    {
        "limit": 25,
        "offset": 0,
        "totalItems": 122,
        "events": [
            {
                "product" : {
      	            "id": "ee013a9b-53cc-45d2-95da-ec50360b8e80",
                },
    	        "event" : "checkout",
	        "time" : "2014-11-17T14:00:09Z"
            },
            { ... }
        ],
        "links": { 
           "previous": [
               { "href" : "http://api.overdrive.com/v1/collections/[collection]/circulation?limit=25&offset=2014-11-17T14:00:09Z" }
           ]
        }
    }

Events we would like to track:

 1. A patron checks out a book, or the system automatically checks a book out 
    on behalf of a patron because they were first in the hold queue.
 2. A book is checked in, either by patron intervention or because its loan
    expires.
 3. A patron puts a hold on a book.
 4. A patron gives up their hold on a book.
 5. The system notifies a patron that a book they put on hold is now
    available.
 6. The system takes a patron out of the hold queue because they never
    responded to the notification in #5.

We are _not_ asking for information about which patron triggered a
given event. In fact, we ask you not to track which patron checked out
a book beyond the term of a checkout.
