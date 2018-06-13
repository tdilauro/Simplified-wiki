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

## Describing availability with `<opds:availability>`

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
+-----------+-----------+--------+---------+

The `opds:availability` tag is OPTIONAL. If it's not present, an OPDS
client MUST assume that the resource at the other end of the
`atom:link` is currently available.

### `opds:state`

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

* `reserved`: The resource is reserved for the authenticated user. The user
  can acquire the resource now, but if they don't act soon, they will lose
  the opportunity.
  
  This is not a binding promise. When the user follows the link, it
  may turn out that the reservation has expired and the resource is now
  unavailable.

### `opds:since` and `opds:until`

The date attributes are OPTIONAL in an `opds:status` tag. They are
used to help the patron plan for the future.

* If `opds:state` is `available`, then `opds:since` is the time at
  which the resource became available, and `opds:until` is the time at
  which the resource will no longer be available.
  
  For example, if the patron has a resource on loan, `opds:since` is the
  loan's start date and `opds:until` is the expiration date.

* If `opds:state` is `unavailable`, then `opds:since` is the last time
  at which the resource was available (probably not useful) and `opds:until`
  is the time at which the resource will become available.
  
  If the patron is in a middle of a hold queue, the library may use
  `opds:until` to provide an _estimated_ time at which the state will
  change to `opds:reserved`. This estimate can't be exact because it
  depends on the behavior of people further up in the hold queue.

* If `opds:state` is `reserved`, then `opds:since` is the time
  at which the resource became reserved for the authenticated user,
  and `opds:until` is the time at which the resource will revert to its
  default availability (probably `unavailable`).





## Holds

## The `borrow` relation
