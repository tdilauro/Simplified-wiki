To bring a new collection of books or other content into a Library
Simplified circulation manager, you'll probably need to do a lot of
work. If the books are made available through OPDS, you can create a
new collection through the admin interface using the "OPDS Import"
protocol. But most of the time you'll need to write code.

Most content sources have no API at all, or have an API that can't be
integrated into Library Simplified
products. [This wiki](ContentSourcesWithoutAPI) has analyses of a
large number of popular library content sources that have no usable
API. If you're in this situation, you'll need to convince the content
provider to make their stuff available in the first place. _Then_
you'll need to write the integration code.

If a content source does offer an API, this is the _minimum_ work
you'll need to do:

* Write a subclass of `Monitor` that keeps track of the items in the
  new collection.

* Write a subclass of `CirculationAPI` to manage the interactions
  between those items and the patrons who want to access them.

On the highest level, you'll need to answer these three questions:

1. How does a library know what titles are in the catalog? (This is the job of the `Monitor` subclass.)
2. How can a library carry out actions on catalog titles on behalf of a patron? (This is the job of the `CirculationAPI` subclass.)
3. Once the patron chooses a title, how is it delivered? (This is also handled by the `CirculationAPI` subclass.)

Depending on the API exposed by the content source, you might need to
write two or three subclasses of `Monitor`.

This high-level document will take you step-by-step through the
process of creating a new content integration.

# Define a license protocol

This part is easy. You're mainly adding constants to the code base so
that the circulation manager knows how to present your content source
as an option to administrators.

First, edit `core/model.py` to add constants to `DataSource` and to
`ExternalIntegration`.

```
class DataSource(Base):
    MYAPI = "My API"

class ExternalIntegration(Base):
    MYAPI = DataSource.MYAPI

    LICENSE_PROTOCOLS = [ ..., MYAPI]
    DATA_SOURCE_FOR_LICENSE_PROTOCOL = {
        ...,
	MYAPI : DataSource.MYAPI,
    }
```

`DataSource` describes a source of information or content. All the
metadata (`Edition`s) and content licenses (`LicensePool`s) that come
from your service needs to be tagged with a corresponding
`DataSource`.

`ExternalIntegration.protocol` is a _technique_ for getting
information or content from some other computer system. The protocol
encapsulates the details of _how_ the circulation manager is supposed
to communicate with your system.

`ExternalIntegration.goal` is the _purpose_ of an integration. Since
the main purpose of your integration is to provide content (as opposed
to metadata, analytics, or something else), your protocol goes into
`LICENSE_PROTOCOLS`.

Since you are integrating a one-off custom API hosted on a particular
server, you are adding both a data source (the server) and a protocol
(the API) to the system. Binding the two inside
`DATA_SOURCE_FOR_LICENSE_PROTOCOL` saves work for someone who's
integrating their circulation manager with your API. They don't have
to specify both the protocol and the data source: the one implies the
other.

# Create a module

Next, create a new module in `circulation/api` named after your
content source. Here are some examples:

* axis.py - Axis 360
* overdrive.py - Overdrive
* oneclick.py - OneClick Digital
* bibliotheca.py - Bibliotheca

Let's say your module is called `myapi.py`. This is where you'll keep
your `CirculationAPI` subclass and your monitors.

# Subclass `CirculationAPI`.

Your `CirculationAPI` subclass abstracts away the details of your API behind a common interface around things a patron might want to do with your collection.

The constructor of your `CirculationAPI` subclass will always be
invoked with a `Collection` object. This object represents someone's
actual collection of titles, obtained through this license
source. You're expected to make all the books in the collection show
up in the circulation manager's database. You're expected to keep all
this information up to date. And whenever a patron wants to get a
title from the collection, you're expected to make it happen.

You'll probably want to subclass `BaseCirculationAPI`, since it contains a lot of logic around turning the patron's desires into calls to your API. These six methods are where the rubber meets the road. You will need to implement all six, even if as a no-op. (If your system has no concept of 'holds', you'll implement the `_hold` methods as no-ops.)

Most of these methods are supposed to return abstract data classes, for which see below.

* `checkin(patron, pin, licensepool)` - The patron wants to cancel their existing loan for `licensepool`. No return value -- it either succeeds or raises an exception.
* `checkout(self, patron, pin, licensepool, internal_format)` - The patron wants to borrow `licensepool`. Returns a `LoanInfo` object.
* `fulfill(self, patron, pin, licensepool, internal_format)` - The patron has already taken out a loan for `licensepool`, and now they want the actual book. Returns a `FulfillmentInfo` object.
* `patron_activity(self, patron, pin)` - The patron wants to see all their current activity. Returns a list of `HoldInfo` and `LoanInfo` objects.
* `place_hold(self, patron, pin, licensepool, notification_email_address)` - The patron wants to place a hold on a title. Returns a `HoldInfo` object. If you do notifications via email when the book becomes availble, `notification_email_address` is the email to send the notification to.
* `release_hold(self, patron, pin, licensepool)` - The patron wants to cancel their hold on a title. No return value -- it either succeeds or raises an exception.

Once you implement your class, add it to the dictionary returned by
`CollectionAPI.default_api_map()`.

```
    def default_api_map(self):
        """When you see a Collection that implements protocol X, instantiate
        API class Y to handle that collection.
        """
	from myapi import MyCirculationAPI
	return {
	    ...,
	    ExternalIntegration.MYAPI : MyCirculationAPI,
	}
```

This ensures that when someone tries to borrow a book from your
collection, `MyCirculationAPI` will be told to handle it, instead of
someone else's API being told to handle it.

Also modify `circulation/api/admin/controller.py` and add your API to the
`provider_apis` list defined in `SettingsController.collections`.

```
        provider_apis = [OPDSImporter,
                         OverdriveAPI,
			 ...
			 MyCirculationAPI,
			]
```

This ensures that administrators will be able to create collections
that get titles from the new content source.

## Abstract data classes

The three classes listed below are defined in
`circulation/api/circulation.py.` All these classes abstract away the
differences between APIs, providing a generic way to store information
about a patron's relationship to your titles.

All of these objects contain the basic information necessary to
identify a specific title:

* The `Collection` the title belongs to.
* The name of the `DataSource` associated with its `LicensePool`.
* The type and identifier of the `Identifier` object associated with its
  `LicensePool`.

### `LoanInfo`

A `LoanInfo` for a title encapsulates information about an a active
loan. In addition to the basic information listed above, which
designates the title on loan, a `LoanInfo` can contain the
following information:

* `start_date` is the time the loan was taken out.
* `end_date` is the time the loan expires.
* `fulfillment_info` is a `FulfillmentInfo` explaining how the patron
  can actually get the book. It's okay to leave this blank--if the
  circulation manager needs a `FulfillmentInfo` and it isn't here,
  it'll call `fulfill()`

## `FulfillmentInfo`

A `FulfillmentInfo` for a title encapsulates information a patron's
client can use to actually download or start reading the book. In
addition to the basic information listed above, which designates the
title being fulfilled, a `FulfillmentInfo` can contain the following
information:

* `content_link`: The URL to the content (e.g. an EPUB) or to a document
  that can be used to get the content (e.g. an ACSM file).
* `content`: An alternative to `content_link`. Instead of providing a
  link to the content, you can provide the actual content, or (more
  likely) a document that can be used to get the content.
* `content_type`: The media type of the `content` or the document at the
  other end of the `content_link`.

* `content_expires`: Access to fulfillment is often time-limited.
  Sometimes the `content_link` will stop working after ten seconds, or
  a minute, or whatever. At that point, if the patron wants the book,
  the circulation manager will need to call `fulfill` again and get a
  brand-new `FulfillmentInfo`. This field contains the time, if it is
  known, when this `FulfillmentInfo` will stop working.

## `HoldInfo`

A `HoldInfo` for a title encapsulates information about a patron's
place in the holds queue for that title. In addition to the basic
information listed above, which designates the title on hold, a
`HoldInfo` can contain the following information:

* `start_date`: The date the patron put the title on hold.
* `end_date`: The _estimated_ date the title will be available for the
  patron to borrow.
* `hold_position`: The patron's current position in the holds queue.
  The smaller this number, the less time the patron will need to wait
  to borrow the title. If this number is zero it means the book is ready
  for the patron to borrow, but has not been borrowed yet.

## Exceptions

The `circulation_exceptions.py` module contains a lot of exceptions
you can raise if something goes wrong during one of the
`CirculationAPI` operations. The circulation manager will catch these
exceptions and either return an appropriate response to the patron, or
try something different. For example, if a patron tries to borrow a
book and your `checkout` implementation raises `NoAvailableCopies`,
we'll call `place_hold` instead and try to put the book on hold.

## Delivery mechanisms

Internally, the circulation manager uses `DeliveryMechanism` objects
to keep track of the formats in which a title is available. But APIs
have their own ways of keeping track of this information. For example,
Overdrive uses the string "ebook-epub-adobe" to refer to an EPUB
document encrypted with Adobe DRM.

If your API makes titles available in multiple formats, you'll need to
define the dictionary `delivery_mechanism_to_internal_format`. It maps
a 2-tuple (media_type, drm_mechanism) to the internal string used in
your API to describe that setup.

The `internal_format` passed in to `checkout` and `fulfill` 

When a title is available in multiple formats, some APIs (like Axis
360) require that the format be chosen at the point of checkout. Other
APIs (like Overdrive) require that the format be chosen the first time
the loan is fulfilled.

The `SET_DELIVERY_MECHANISM_AT` constant lets you specify whether your
API is a "choose at point of checkout" API, in which case the chosen
`internal_format` will be passed into `checkout()`, or a "choose at
first fulfillment" API, in which case the chosen `internal_format`
will be passed into `fulfill`. The default is to choose the format
during first fulfillment.

# Add configuration settings

At some point, your `CirculationAPI` subclass needs to actually make
HTTP requests (or whatever) to the API that has the content.

The problem is, that API probably isn't accessible to everyone in the
world. The API provider has rules about who can access it and who
can't. They give out API keys or passwords. When an HTTP request comes
in trying to borrow a book from a certain collection, that API has
some way of knowing that the request is authorized by the entity who
paid for that collection, and it's not some random person trying to
scam a free book.

In short, APIs need to be _configured_ with IDs, API keys, access
URLs, and so on. You'll need to specify up front what kind of
configuration settings the content integration expects. A library
administrator can find these values from the API provider and fill
them in through the administrative interface.

Let's take a look at the Overdrive configuration
(`circulation/api/overdrive.py') for an example.

```
class OverdriveAPI(BaseOverdriveAPI, BaseCirculationAPI):
    NAME = ExternalIntegration.OVERDRIVE
    SETTINGS = [
        { "key": Collection.EXTERNAL_ACCOUNT_ID_KEY, "label": _("Library ID") },
        { "key": BaseOverdriveAPI.WEBSITE_ID, "label": _("Website ID") },
        { "key": ExternalIntegration.USERNAME, "label": _("Client Key") },
        { "key": ExternalIntegration.PASSWORD, "label": _("Client Secret") },
    ] + BaseCirculationAPI.SETTINGS
```

In addition to the settings common to all integrations of this type
(`BaseCirculationAPI.SETTINGS`), the Overdrive configuration defines
four settings that are provided when a library signs up for access to
the Overdrive API. What they do exactly doesn't matter for purposes of
this discussion: all four are necessary to access the Overdrive API,
so we need the administrator to provide values for all four.

We try to put the Overdrive settings into standard locations (we store
Overdrive's "client key" in the standard setting called
`ExternalIntegration.USERNAME`, because it's basically a username),
but you can call your configuration settings whatever you want.

Once you set this up, you should be able to use the administrative
interface to create a collection for the new content source and
configure it to connect to the content source's API.

When your `BaseCirculationAPI` subclass is instantiated, the
`collection` object that's passed in will have an
`.external_integration` object associated with it. You can access its
configuration values by writing code like this:

```
integration = overdrive_collection.external_integration
website_id = integration.setting(BaseOverdriveAPI.WEBSITE_ID).value
```

This should give you everything you need to make HTTP requests to the
API. Unfortunately, you can't deliver books to patrons, because you
don't yet have any way to actually get titles into the system from the
API.

# The `Monitor` subclass

A monitor is a piece of code that runs on a regular basis. Your
monitor is responsible for making sure that the circulation manager's
local picture of a collection reflects the actual collection on the remote
side.

Your monitor needs to know every time one of these events happens:

* A book becomes available.
* A book stops being available altogether.

When you find out about a new book, you need to grab as much
bibliographic information about that book as possible and get it into
the system. The simplest way to do this is through the [[metadata layer|MetadataLayer]]. When you find out that a book is no longer
available, you'll need to modify its `LicensePool` to reflect this
fact. The metadata layer can help with this, too.

For collections that offer a limited number of simultaneous uses of a
title (as opposed to metered access or some other mechanism), your
monitor must also know when these things happen:

* The number of available licenses for a title goes up or down (e.g. because someone borrowed or returned it).
* The number of patrons in a hold queue changes (e.g. because someone put a book on hold or cancelled their hold).
* Licenses are added to or removed from a title.

Any of these changes must result in a corresponding change to the
title's `LicensePool`.

Your basic monitor is going to be a subclass of
`CollectionMonitor`. It will take a `Collection` object in its
constructor, and its job is to bring the local model of that
collection up to date with the source of truth on the other end of the
network.

There are three basic strategies for `CollectionMonitor`s:

* A "delta" type monitor which runs very frequently and only asks
  about changes to the collection since the last time it ran. There's
  no superclass for this, but you can see
  `OverdriveCirculationMonitor` or `BibliothecaEventMonitor` for
  examples.

* A "sweep" type monitor which asks about the current status of every
  single item known to be in the collection. (To implement this,
  subclass `IdentifierSweepMonitor` or `WorkSweepMonitor`).

  This is a good technique for finding changes that slipped through
  the cracks, but it can't detect when items are added to the
  collection, because it only operates on items already known to be in
  the collection.

* An "everything" monitor which gets a complete listing of the
  collection every single time it's run. You might use this to find
  out about new and removed titles, leaving it to a "sweep" monitor to
  follow up and find details. There's no superclass for this, but you
  can see `OneClickCirculationMonitor` for an example.

We've learned through experience that even a content source that
offers a delta API probably also needs a "sweep" monitor or an
"everything" monitor to occasionally check on the collection as a
whole. A delta API is a lot more efficient on a day-to-day basis, but
you can't assume you always hear about every single change.

# Cron jobs

TBD
