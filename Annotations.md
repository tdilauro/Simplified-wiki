Our current goal is to sync a patron's current reading position across devices. Our desiderata:

* The same system should eventually support bookmarks, highlights, notes and annotations.
* The system needs to be extensible to allow annotations on 
* Reuse existing specs whenever possible. Sacrifice simplicity for reusability/interoperability.

## Web Annotation Protocol

Our core protocol is the [Web Annotation Protocol](https://www.w3.org/TR/annotation-protocol/), a simple REST wrapper around the [Linked Data Platform](https://www.w3.org/TR/ldp/). Its data model is taken from the [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/) and [Activity Streams Collections](https://www.w3.org/TR/activitystreams-core/#collections).

For EPUB books in particular, we need to take vocabulary from [Open Annotation In EPUB](http://www.idpf.org/epub/oa/). We are _not_ adopting Open Annotation In EPUB wholesale because 1) it's EPUB-specific, 2) it's not a web-based protocol but rather a set of rules for storing annotations _inside_ an EPUB. Since patrons don't generally get to keep the EPUBs they borrow from the library, that's not an option.

All of these specs are based on [JSON-LD](https://www.w3.org/TR/json-ld/). The Web Annotation Protocol is the concrete implementation of what is described in abstract terms by [Portable Web Publications for the Open Web Platform](https://www.w3.org/TR/pwp/).

## Minimal Viable Product

1. Your bookshelf should link to your annotations feed.
2. Sending GET to your annotations feed should give you a complete list of all your current reading positions.
3. POSTing an annotation to your annotations feed will add it to the list.
4. An annotation will be rejected unless it is the current reading position for a book for which the patron has an active loan. However, annotations are associated with identifiers, not with loans, because they generally need to outlive loans.
5. Each annotation position has a permalink and sending DELETE to that link will remove the annotation.
6. A given loan can only have one current reading position for a given patron. A new reading position overwrites  an old one.
7. When the patron returns a book their current reading position for that book is at risk, but is not immediately deleted. (Because they might borrow the book again in the near future.)