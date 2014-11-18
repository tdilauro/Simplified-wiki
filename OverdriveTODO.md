## Overdrive requirements document (sketch)

### DRM credential management

We currently have no visibility into the DRM credentials associated with a patron's account, whether they were created by Overdrive or created by the patron and manually associated with the patron's Overdrive account.



### Circulation tracking

We currently use two techniques for tracking the circulation of our
Overdrive materials:

1. Every minute, we invoke the `search` API to find all the records
   that have changed since since the last minute. We use the library
   availability API to check on the availability for each book
   mentioned.

   In theory, this is all we need, except for a one-time crawl to
   establish the initial collection. In practice we have found that
   this technique misses a lot of circulation events, so we use the
   second technique to fill in the gaps.

2. We use the `products` link to crawl our entire collection, using
   the library availability API to check on availability. A complete
   crawl takes approximately 4 hours, though by parallelizing the
   library availability calls we could speed this up considerably.

Even with both systems in place there are events we don't see. For
example, when a book is checked in and then immediately checked out to
the next patron in the hold queue, the only thing we see is the number
of holds decreasing by one.

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
