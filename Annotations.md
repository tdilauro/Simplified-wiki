Our current goal is to sync a patron's current reading position across devices. Our desiderata:

* The same system should eventually support bookmarks, highlights, notes and annotations.
* The system needs to be extensible to work with other media types: PDF, HTML, plain text.
* Reuse existing specs whenever possible. Sacrifice simplicity for reusability/interoperability.

## Web Annotation Protocol

Our core protocol is the [Web Annotation Protocol](https://www.w3.org/TR/annotation-protocol/), a simple REST wrapper around the [Linked Data Platform](https://www.w3.org/TR/ldp/). Its data model is taken from the [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/) and [Activity Streams Collections](https://www.w3.org/TR/activitystreams-core/#collections).

There is a [Web Annotation Protocol test client](https://github.com/BigBlueHat/web-annotation-protocol-tester).

For EPUB books in particular, we need to take vocabulary from [Open Annotation In EPUB](http://www.idpf.org/epub/oa/). We are _not_ adopting Open Annotation In EPUB wholesale because 1) it's EPUB-specific, 2) it's not a web-based protocol but rather a set of rules for storing annotations _inside_ an EPUB. Since patrons don't generally get to keep the EPUBs they borrow from the library, that's not an option.

All of these specs are based on [JSON-LD](https://www.w3.org/TR/json-ld/).

[Portable Web Publications for the Open Web Platform](https://www.w3.org/TR/pwp/) is a related document that we're more or less ignoring because it doesn't have any practical application ATM.

## Minimal Viable Product

1. Your bookshelf links to your annotations feed.
2. Sending GET to your annotations feed gives you a complete list of all your current reading positions.
3. POSTing an annotation to your annotations feed adds it to the list.
4. An annotation will be rejected unless its motivation is "idling" (i.e. leaving the book to come back to it later).
5. An annotation will be rejected unless it is associated with an identifier for which the patron has an active loan.
6. Each annotation position has a permalink and sending DELETE to that link will remove the annotation.
7. A given loan can only have one current reading position for a given patron. A new reading position overwrites  an old one.
8. When the patron returns a book their current reading position for that book is at risk, but is not immediately deleted. (Because they might borrow the book again in the near future.)

## Possible database schema

```
annotations
id
patron_id
identifier_id
motivation
timestamp
active (boolean)
content
```

Annotations are associated with identifiers, not with loans, because they generally need to outlive loans.

An annotation that is deleted has `active` set to False. Its `content` is blanked out and its `timestamp` is updated.

`motivation` corresponds to the Web Annotation Data Model's idea of "motivation" and will be used to group annotations of different types (e.g. page bookmark vs current reading position or highlighted passage). Note that under WADM, an annotation MAY have zero or many motivations, but SHOULD have one.

`timestamp` is set when the annotation is created and updated whenever it is updated.

## The link

According to [Discovery of annotation containers](*https://www.w3.org/TR/annotation-protocol/#discovery-of-annotation-containers), a resource advertises an annotation container using the `Link` header and the link relation `http://www.w3.org/ns/oa#annotationService`.

```
Link: <http://example.org/annotations/>; rel="http://www.w3.org/ns/oa#annotationService"
```

## Retrieving the annotation container

Covered in [Container Retrieval](https://www.w3.org/TR/annotation-protocol/#container-retrieval)

```
GET /annotations/
Prefer: return=representation;include="http://www.w3.org/ns/oa#PreferContainedDescriptions"
```

There are a number of different rules for getting data at different levels of granularity. To start with we only need to support `PreferContainedDescriptions`.

## Representation of annotation containers

The container data model is described in [Annotation Collection](https://www.w3.org/TR/annotation-model/#annotation-collection). Its media type is `application/ld+json; profile="http://www.w3.org/ns/anno.jsonld"`. 

Our representations will need to include the JSON-LD context for Open Annotation in EPUB (`http://www.idpf.org/epub/oa/1.0/context.json`). Since "idling" is a motivation we are making up, we also need to include our own custom context.

### An empty container

This is just a guess at what an empty container would look like.

```
HTTP/1.1 200 OK
Content-Type: 
Allow: GET,OPTIONS,HEAD
Vary: Accept, Prefer
Content-Length: 924

{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/annotations/,
  "type": "AnnotationPage",
  "partOf": {
    "id": "http://example.org/annotations/",
    "total": 0
  },
  "items": [
  ]
}
```

### A container with items

Taken from [Responses With Annotations](https://www.w3.org/TR/annotation-protocol/#responses-with-annotations)

```
HTTP/1.1 200 OK
Content-Type: application/ld+json; profile="http://www.w3.org/ns/anno.jsonld"
Allow: GET,OPTIONS,HEAD
Vary: Accept, Prefer
Content-Length: 924

{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/annotations/?iris=0&page=0",
  "type": "AnnotationPage",
  "partOf": {
    "id": "http://example.org/annotations/?iris=0",
    "total": 42023
  },
  "next": "http://example.org/annotations/?iris=0&page=1",
  "items": [
    {
      "id": "http://example.org/annotations/anno1",
      "type": "Annotation",
      "body": "http://example.net/body1",
      "target": "http://example.com/page1"
    },
    {
      "id": "http://example.org/annotations/anno2",
      "type": "Annotation",
      "body": {
        "type": "TextualBody",
        "value": "I like this!"
      },
      "target": "http://example.com/book1"
    }
    // ...
    {
      "id": "http://example.org/annotations/anno50",
      "type": "Annotation",
      "body" : "http://example.org/texts/description1",
      "target": "http://example.com/images/image1"
    }
  ]
}
```

Note the pagination feature, which we won't be dealing with yet.

## Parts of an annotation

Here's a simple example taken from the [Web Annotation Data Model](https://www.w3.org/TR/annotation-model/#motivation-and-purpose). For our purposes, an annotation has a target, a body, and a motivation.

```
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "http://example.org/anno18",
  "type": "Annotation",
  "motivation": "bookmarking",
  "body": [
    {
      "type": "TextualBody",
      "value": "A good description of the topic that bears further investigation",
      "purpose": "describing"
    }
  ],
  "target": "http://example.com/page1"
}
```

The _target_ is the thing being annotated. In our case it will either be a book, a position in a book, or a span in a book. For MVP it will always be a position in a book.

The _motivation_ is why the patron created the annotation. This is how we distinguish between idling, bookmarking, and highlighting. For MVP this will always be 'idling'.

The _body_ is the content of the annotation. For MVP this will never be present, and the server does not have to care about it.

Note that (Open Annotation in EPUB)[http://www.idpf.org/epub/oa/] uses different terms from Web Annotation Data Model: it uses `hasTarget`, `hasBody`, `motivatedBy`; WADM has `target`, `body`, `motivation`.  Open Annotation in EPUB is based on (Open Annotation)[http://www.openannotation.org/spec/core/], which was made obsolete by the Web Annotation Data Model. We will be using WADM except when OAiEPUB has a concept we need that is not defined in WADM.

### Target format

Web Annotation Data Model defines various ways of targeting part of an HTML document, using [selectors](https://www.w3.org/TR/annotation-model/#selectors). Open Annotation in EPUB [defines](http://www.idpf.org/epub/oa/#id.eso6b8nzsvsp) a way to use Canonical Fragment Identifiers to target part of an EPUB.

A target has a `source`, which in our case identifies the book, and a `selector`, which identifies part of a book. For `source` we'll be using the URN for a book's identifier. This is the same URN seen in the `<id>` tag of an OPDS entry.

We can borrow `FragmentSelector` from Open Annotation in EPUB. CFI has a lot of problems, so we probably won't use this. But here's what it would look like if CFI were reliable enough to use:

```
"target": {
    "source": "urn:librarysimplified.org/terms/id/3M%20ID/c73g2z9",
    "selector": {
      "type": "oa:FragmentSelector",
      "value": "epubcfi(/6/4[chap01ref]!/4[body01]/10[para05]/3:10)"
    }
}
```

We will most likely develop our own custom selector format to replace `FragmentSelector` with an alternate way of identifying a resource within an EPUB document. We can then use [Refinement of Selection](https://www.w3.org/TR/annotation-model/#refinement-of-selection) to combine our custom selector with [some other selector](https://www.w3.org/TR/annotation-model/#selectors).

Hadrien mentions that we may want to specify multiple selectors: multiple redundant paths into the same part of the resource.