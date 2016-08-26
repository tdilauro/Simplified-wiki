Our current goal is to sync a patron's current reading position across devices. Our desiderata:

* The same system should eventually support bookmarks, highlights, notes and annotations.
* The system needs to be extensible to allow annotations on 
* Reuse existing specs whenever possible. Sacrifice simplicity for reusability/interoperability.

== Web Annotation Protocol

Our core protocol is the [Web Annotation Protocol](https://www.w3.org/TR/annotation-protocol/), a simple REST wrapper around the [Linked Data Platform](https://www.w3.org/TR/ldp/). Its data model is taken from the [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/) and [Activity Streams Collections](https://www.w3.org/TR/activitystreams-core/#collections).

For EPUB books in particular, we need to take vocabulary from [Open Annotation In EPUB](http://www.idpf.org/epub/oa/). We are _not_ adopting Open Annotation In EPUB wholesale because 1) it's EPUB-specific, 2) it's not a web-based protocol but rather a set of rules for storing annotations _inside_ an EPUB. Since patrons don't keep most of the EPUBs they borrow from the library, that's not an option.