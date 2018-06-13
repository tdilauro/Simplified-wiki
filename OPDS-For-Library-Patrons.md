# OPDS For Library Patrons

## Motivation

Unlike electronic bookstores and open-access repositories, public
libraries often license ebooks for their patrons under terms that
impose an artificial scarcity. Any of these scenarios may apply to a
public library trying to get an ebook to one of its patrons:

* The library has licensed a certain number of 'copies' of a book, and
  all the 'copies' are currently out on loan.
* The library has a daily budget for loaning out books, which has been
  exceeded. No more titles from this vendor may be loaned out today.
* The library may only deliver a book to a patron as a time-limited
  loan. After the loan expires, the patron is obliged to erase their
  local copy of the book.

As a response to the first scenario (not enough 'copies'), some
electronic libraries have introduced the concept of _holds_ from the
physical world of branch libraries. A hold is a promise from library
to patron that the patron will get a chance to borrow a book
_eventually_, once other patrons in the 'hold queue' get a chance to
read the book.

This specification documents OPDS extensions that allow public
libraries to communicate these scenarios of artificial scarcity --
availability, holds queues, loan durations -- to OPDS clients. It also
sets out guidelines for how an OPDS client should process the standard
`http://opds-spec.org/acquisition/borrow` link relation.

## `opds:availability` - Describing resource availability

OPDS operates on the background assumption that a book with an OPDS
entry is available right now and will always be available. In a
public library environment, this is generally not true:

* The patron might have a time-limited loan for a book. It's available
  now, but soon it won't be.
* The patron might be in the middle of a hold queue. The book is not
  available now, but it will be eventually.
* The patron might be at the _front_ of a hold queue. The book is
  available now, but if the patron doesn't borrow it soon, they will
  lose their opportunity and have to go to the back of the queue.

Even in online bookstores, the 'now and always available' assumption
does not necessarily hold. Two possible scenarios:

* A book may be available for pre-order. The user can buy the book
  now, but the purchase will not be fulfillable until a certain date.
* A book may be not be available after a certain date due to an
  expiring license, analogous to how films leave Netflix.

The `opds:availability` tag goes inside an `atom:link` tag and
communicates the current availability of the resource at the other end
of the link.

Three attributes are defined for the `opds:availability` tag:

| Attribute | Semantics | Values | Default |
| --------- | --------- | ------ | ------- |
| state     | The current availability state of the resource. | Four values for this attribute are defined below: `available`, `unavailable`, `reserved`, and `ready`. | `available` |
| since     | Date when the availability state changed to the current state. | ISO 8601 date | No value |
| until     | Date when the availability state is expected to change again. | ISO 8601 date | No value |

The `opds:availability` tag is OPTIONAL. If it's not present, an OPDS
client MUST assume that the resource at the other end of the
`atom:link` is currently available.

### `opds:state` - Can I have this book?

The `opds:state` attribute is REQUIRED for an `opds:availability` tag.

These values are defined for `opds:state`:

* `available`: The resource is available right now. If the user is
  authenticated, this means the resource is available _to that user_ -- it
  might not be available generally. If the user is not authenticated,
  this means that any authenticated user can expect to be able to
  obtain the resource.
  
  This is not a binding promise. When the user follows the link, the resource
  may turn out not to be available after all.

* `unavailable`: The resource is not currently available. The user can
  still follow the acquisition link, but this will result in some
  intermediate step, such as a preorder or a hold, rather than the
  immediate acquisition of the resource.
  
  This is not a binding promise. When the user follows the link, the
  resource may turn out to be available after all.

* `reserved`: The authenticated user will get the resource eventually,
  but it's not available now. This generally means the authenticated
  user has placed a hold on the resource and is waiting in a holds
  queue.

* `ready`: The resource is ready for the authenticated user, but not
  for the general public. The user can acquire the resource now, but
  if they don't act soon, they will lose the opportunity.
  
  This is not a binding promise. When the user follows the link, it
  may turn out that the reservation has expired and the resource is now
  unavailable.

### `opds:since` and `opds:until` - When does my loan expire?

The date attributes are OPTIONAL in an `opds:status` tag. They are
used to help the patron plan for the future.

* If `opds:state` is `available`, then `opds:since` is the time at
  which the resource became available, and `opds:until` is the time at
  which the resource will no longer be available.
  
  For example, if the patron has a resource on loan, `opds:since` is the
  loan's start date and `opds:until` is the expiration date.

* If `opds:state` is `unavailable`, then `opds:since` is the last time
  at which the resource was available (probably not useful) and
  `opds:until` is the estimated time at which the resource will become
  available. The `opds:until` time may be conditional on the user
  taking some immediate action, such as putting a hold on the
  resource.

* If `opds:state` is `reserved`, then `opds:since` is the time at
  which the authenticated user placed their hold, and `opds:until` is
  the _estimated_ time at which the resource will become available
  (probably by moving into the `ready` state). This estimate can't be
  exact because it depends on the behavior of people further up in the
  hold queue.
  
  If the patron does not have an active loan, then `opds:until` for an
  `unavailable` resource may indicate the estimated time at which the
  resource _will_ become available if the patron joins the hold queue now,

* If `opds:state` is `ready`, then `opds:since` is the time
  at which the resource became ready to the authenticated user,
  and `opds:until` is the time at which the resource will revert to its
  default availability (probably `unavailable`).

## `opds:copies` - How many copies does the library have?

The `opds:copies` tag describes the number of licenses the server owns
for a resource. Although this is intended to describe electronic
licenses for virtual resources, it may also be used to represent
physical books in a bookstore or branch library.

Like `opds:availability`, `opds:copies` goes inside an `atom:link` tag
and describes the resource at the other end of the link. If multiple
links give access to the same resource (e.g. in different formats),
each link SHOULD provide the same information in its `opds:copies`.

`opds:copies` has two optional attributes, `opds:total` and
`opds:available`. Both have a numeric value.

* `opds:total` is the total number of licenses available to the server.
* `opds:available` is the number of those licenses that are available
  to patrons right now (e.g. not disabled or on loan to specific patrons).

If the `opds:state` is `unavailable`, then `opds:available` SHOULD be `0`.

## `opds:holds` - Where am I in line?

The `opds:holds` tag describes the people waiting in line for access
to a resource.

Like `opds:availability`, `opds:holds` goes inside an `atom:link` tag
and describes the resource at the other end of the link. If multiple
links give access to the same resource (e.g. in different formats),
each link SHOULD provide the same information in its `opds:holds`.

`opds:holds` has two optional attributes, `opds:total` and
`opds:position`. Both have a numeric value.

* `opds:total` Is the total number of people waiting for access to this
  resource.
* `opds:position` is the position in that queue of the currently
  authenticated user.

If the `opds:state` is `available`, then `opds:total` SHOULD be `0`.

## Examples

This example shows a book which is not currently available, but which
can be put on hold (the first link, with the relation
`http://opds-spec.org/acquisition/borrow`) or pre-ordered (the second
link, with the relation `http://opds-spec.org/acquisition/buy`).

```
<entry>

 <link type="application/atom+xml;type=entry;profile=opds-catalog" rel="http://opds-spec.org/acquisition/borrow" href="http://example.org/hold/1">
   <opds:availability state="unavailable" until="2019-09-07"/>
   <opds:indirectAcquisition type="application/vnd.adobe.adept+xml">
      <opds:indirectAcquisition type="application/epub+zip"/>
   </opds:indirectAcquisition>
 </link>

 <link type="text/html" rel="http://opds-spec.org/acquisition/buy" href="http://example.org/buy/1">
   <opds:price currency="EUR">10.99</opds:price>
   <opds:availability state="unavailable" until="2019-09-07"/>
   <opds:indirectAcquisition type="application/vnd.adobe.adept+xml">
      <opds:indirectAcquisition type="application/epub+zip"/>
   </opds:indirectAcquisition>
 </link>
</entry>
```

In this example, there are 100 people in the hold queue, 20 copies in
total (none of them available) and the user has yet to place a hold
for the book. The available state is `unavailable` and the `opds:until` 
attribute is an estimate of how long the user will be waiting in the
hold queue.

```
<link ...>
  <opds:availability state="unavailable" until="2019-09-07"/>
  <opds:indirectAcquisition type="application/epub+zip"/>
  <opds:holds total="100"/>
  <opds:copies total="20" available="0"/>
</link>
```

In this example, there are 93 people in the hold queue for a book, and
87 of those people are ahead of the authenticated user. The
availability state is `reserved` and the `opds:until` attribute gives
the estimated time at which the book will be available to this user.

```
<link ...>
  <opds:availability state="reserved" until="2015-09-07"/>
  <opds:indirectAcquisition type="application/epub+zip"/>
  <opds:holds total="93" position="88"/>
  <opds:copies total="19" available="0"/>
</link>
```

In this example, the authenticated user is at the front of the hold
queue but has not yet acquired a loan. The `opds:position` element is
missing from `opds:holds`. The `opds:availability` element indicates
that the user can get the book now (`opds:state` is `ready`) but that
if they don't act, they eventually lose their opportunity and have to
place a new hold (`opds:until` gives the time at which this will
happen).

```
<link ...>
  <opds:availability state="ready" since="2018-09-07" until="2018-09-10"/>
  <opds:holds total="59"/>
  <opds:copies total="19" available="0"/>
</link>
```

In this example, the user has acquired a loan for the book. Here the
`opds:availability` element is most important. Its `opds:state`
attribute is `availabile`. The `opds:since` attribute shows the time
at which the loan was created, and the `opds:until` attribute shows
the time at which the loan will expire.

```
<link ...>
  <opds:availability state="available" since="2018-09-09" until="2018-10-09"/>
  <opds:holds total="58"/>
  <opds:copies total="19" available="0"/>
</link>
```


## The `borrow` relation
