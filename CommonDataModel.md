Library Simplified Server-side Data Model Overview



This data model is common between the circulation manager and the metadata wrangler, although some pieces are exclusively used by one component or the other. For example, only the circulation manager has lanes or patrons, and only the metadata wrangler has integration clients. The library registry component has a separate data model which is similar but much simpler.

Although the data model is very complex, it can be split up into six relatively simple systems:

* Bibliographic metadata
* Hyperlinks
*
*
*
*

For the sake of simplicity, this document will talk about "books", but the rules are the same for audiobooks and other forms of content.

# Bibliographic metadata

Bibliographic information is information _about_ books as opposed to the books themselves. A book's title, its cover image, and its ISBN are all bibliographic information--the text of the book is not. Bibliographic information flows into the circulation manager and metadata wrangler from a variety of sources, mainly OPDS feeds and proprietary APIs. We keep track of all this information and where it came from, and when necessary we weigh it, sort it, and boil it down into a small amount of information that can be used by other parts of the system.

## `Identifier`

An `Identifier` provides a way to uniquely refer to a particular book. Common types of `Identifier` include ISBNs and proprietary IDs such as Overdrive or Bibliotheca IDs.

An `Identifier` may:

* Have many `Classification`s representing how the book would be shelved in a bookstore or library. (See the classification subsystem.)
* Have many `Measurement`s of quantities like quality and popularity. (See the measurement subsystem.)
* Have many `HyperLink`s to associated files such as cover images or descriptions. (See the linked resources subsystem.)
* Participate in many `Equivalency`s.
* Serve as the `primary_identifier` for multiple `Edition`s.
* Serve as the `identifier` for many `LicensePool`s, through `Collection`.
* Be associated with one Work, through Edition
  

### `Equivalency`

An `Equivalency` is an assertion made by a `DataSource` that two different `Identifiers` refer to the same book.

* The `strength` of the `Equivalency` is a number from -1 to 1 indicating how much we trust the assertion. When Overdrive says that an Overdrive ID is equivalent to an ISBN, we give that `Equivalency` a `strength` of 1, because Overdrive got the ISBN from the publisher and assigned the Overdrive ID itself. When OCLC says that two ISBNs represent the same book, we give it a lower `strength`, because OCLC is frequently wrong about this. A negative `strength` means that the `DataSource` is pretty sure two `Identifier`s represent _different_ books.

## `Edition`

An `Edition` is a collection of information about a book from a particular data source. Like most items in the "bibliographic metadata" section, it represents an _opinion_. If different data sources give conflicting information about a book, that's fine -- everyone has their opinion. When this happens, we create multiple `Edition`s and we sort it out later, when it's time to make the _presentation edition_.

An `Edition`:

* Has one `DataSource`. This is the data source whose opinions are recorded in the `Edition`.
* Has one `Identifier`, the `primary_identifier`. This identifies the book the data source is talking about.
* Contains basic metadata -- title, series, language, publisher, medium -- for that book.
* May have one or more `Contributor`s, through `Contribution`.
* May be the _presentation edition_ for a specific `Work`. The presentation edition is a synthetic `Edition` created by the system. We look over a bunch of `Edition`s which are all (supposedly) talking about the same book, and consolidate it into a new `Edition` containing the best or most trusted metadata.

## The contributor subsystem

This system basically tracks who wrote which book. There are two classes in this subsystem: `Contributor` and `Contribution`. 

### `Contributor`

A `Contributor` is a human being or a corporate entity who is credited with work on some `Edition`. The credit itself is kept in a `Contribution`, which ties a `Contributor` to an `Edition`.

A `Contributor`:

* Contains basic biographical information about a person or corporation. Most notably, it has both a `display_name` such as "Octavia Butler", the name that would go on the front of a book, and a `sort_name` such as "Butler, Octavia", the name that would go in a card catalog.

### `Contribution`

A `Contribution`:

* Links a `Contributor` to an `Edition`.
* Contains a `role` describing the work the `Contributor` did on the `Edition`. Common roles include author, editor, translator, illustrator, and narrator.

## The classification subsystem

This system tracks how a book might be classified in a card catalog or shelved in a bookstore. There are two classes in this subsystem: `Subject` and `Classification`.

### `Subject`

A `Subject` represents a classification that someone might give a book. `Subject` handles a variety of classification schemes: Dewey Decimal, LLC, LCSH, BISAC, proprietary systems like Overdrive's, and free-form tags, among others. Four pieces of information might be stored with the `Subject`:

* Genre ("Billionare Romance" is a type of romance)
* Fiction/nonfiction status ("Science Fiction" is always fiction)
* Target audience ("Young Adult Fantasy" is always YA)
* Target age ("Picture books" are generally for very young children, not 12-year-olds.)

### `Classification`

A `Classification` is someone's opinion that a book should be filed under a certain `Subject`.

A `Classification`:

* Links a `Subject` to an `Identifier`.
* Has an associated `DataSource` -- this tracks whose opinion it is.
* Has an associated `weight` representing how certain we are that the book should be filed under this subject. The higher the number, the more certain we are. If OCLC says that a single library has filed a certain book under "Whales", we'll record that information but give it a low `weight`. If OCLC says that ten thousand libraries have filed this book under "Whales", then it's probably about whales.

## The measurement subsystem

## The linked resources subsystem

WORKS:

A Work represents a book in general, as opposed to one specific edition of that book.  A Work:
*May have copies scattered across many LicensePools
*May have many Editions, but derives its presentation metadata from one particular Edition, which is known as its “presentation edition.”  Each LicensePool associated with the Work has its own presentation edition; the highest-quality of these is set as the Work’s default presentation edition and displayed to patrons.
*Stores information about the work’s subject matter classification, intended audience, and popularity, the best available summary for it, and whether it is fiction.
*May be referenced by multiple CustomListEntries and/or CachedFeeds
*May participate in many WorkGenre assignments (i.e. assignments of a Genre to a Work).  






DATASOURCES:

A DataSource provides information and bibliographic metadata about, potentially including licenses for, a book.  For example, a DataSource may provide the information that is used to determine the book’s contributors, subject, and/or measurements.  In some cases, a DataSource may also provide the book’s actual content.  Examples of well-known DataSources include Project Gutenberg, Overdrive, Amazon, and the New York Times.  A DataSource may also:
*Have one IntegrationClient
*Generate many Editions
*Generate many CoverageRecords
*Generate many Equivalencies
*Grant access to many LicensePools
*Provide many Hyperlinks
*Provide many Resources
*Provide many Classifications
*Have many associated Credentials
*Generate many CustomLists
*Provide many LicensePoolDeliveryMechanisms


REPRESENTATIONS and RESOURCES:

A Representation is a cached document obtained from the Web.  They can be associated with  Identifiers via Resources.  For example, in the case of a cover image for a book: a Hyperlink would be created to associate the book’s Identifier with a particular URL (the Resource), which would lead to an image file (the Representation).  

If a Resource is a derivative of another Resource, a ResourceTransformation object is created as a record of this.  The ResourceTransformation object stores the original Resource, the derived Resource, and the settings that were used to transform the former into the latter.



PATRONS:

Each Patron belongs to one Library.  (The human being represented by the Patron object may, in real life, patronize multiple libraries, but will have a separate patron account at each one.)  

A Patron can have:
    * Loans: books which are currently checked out by that patron
    * Holds: books which that patron is in line to check out
    * Annotations: bookmarks, comments, etc. which the patron has created for                  particular books.  

All of these are created via the LicensePool associated with the collection which contains the book in question.  

A Patron can also have: 
    * Complaints against a LicensePool.  Patrons lodge one or more Complaints against a specific LicensePool.  The purpose of Complaints is to report problems pertaining to         specific books; for example, a Patron can lodge a Complaint stating that a book is incorrectly     categorized or described, or that there is a problem with checking it out, reading, or returning it. 

LIBRARIES:

Each Library can have: 
    * one or more Collections.  A Collection is a set of LicensePools, through which books are provided to the Library.  Each Collection can have multiple Libraries to which it provides books, and can have multiple child Collections and multiple LicensePools.      
    * one or more CustomLists.  A CustomList is a list of books, typically grouped by a criterion such as genre, subject, bestseller status, etc., which a librarian has compiled in the admin interface.  Each CustomList is associated with, and presented to patrons in the front-end as, one Lane.  A CustomList has at least one CustomListEntry, each of which refers to a particular Work.      
    * one or more Lanes, each of which is associated with one CachedFeed.
    * one or more Admins.  Admins are people who have access to the admin interface (via accounts in the circulation manager), such as librarians.   They are associated with a particular library, via AdminRole.  An Admin may have more than one AdminRole.  The potential AdminRoles are: SystemAdmin; SitewideLibraryManager; LibraryManager; SitewideLibrarian; and Librarian
    


LICENSING:

A LicensePool is a group of licenses granting access to one particular Work.  If a Work is not associated with a LicensePool, patrons will not be able to check it out.  In some cases, usually involving open-access LicensePools, there may be more than one LicensePool associated with the same Work; if this happens, the LicensePool which provides the highest-quality version of the book will take precedence.  Each LicensePool:
*is associated with the Identifier and the DataSource of the Work to which it grants access
*belongs to one Collection
*has one Edition, containing the metadata used to describe the Work
*can have many Loans, Holds, Annotations, and Complaints
*can have many CirculationEvents.  A CirculationEvent is a record of a change to the LicensePool’s circulation status.  Types of CirculationEvent include events taking place within the circulation manager (e.g. works being checked out or placed on hold), events reported by a distributor (such as licenses being added or removed), and events reported by a client app (i.e. a book having been opened).
*has at least one DeliveryMechanism, through LicensePoolDeliveryMechanism.  A DeliveryMechanism is the means by which a distributor delivers a book to a Patron.  There are two parts to a DeliveryMechanism: 1) the DRM scheme implemented by the distributor, and 2) the content type of the book (e.g. Kindle, Nook, etc.).
*has a RightsStatus, through LicensePoolDeliveryMechanism.  A RightsStatus represents the terms under which a book has been made available to the public. The most common varieties of RightsStatus are 1) in copyright, 2) public domain, and 3) a Creative Commons license. 

CREDENTIALS:

A Credential object stores one Patron’s credentials for external services.  One major example of a type of credential is a Patron’s “Identifier for Adobe Account ID purposes” Credential.  A Credential may have:
*many associated DRMDeviceIdentifiers.  A DRMDeviceIdentifier registers the Patron’s device with a particular DRM scheme.
*one associated DelegatedPatronIdentifier.  A DelegatedPatronIdentifier is an identifier that this library has generated for a patron of a different library.





BACKGROUND PROCESSES:

* A Timestamp provides a record of when a Monitor was run.
* A CoverageRecord provides a record of any processes that have been performed on a book (referred to via its Identifier)
* A WorkCoverageRecord provides a record of any processes that have been performed on a Work (similar to what CoverageRecord does for Identifiers).

CONFIGURATION:

A ConfigurationSetting  holds information about an extra piece of site configuration.  A ConfigurationSetting may be associated with an ExternalIntegration, a Library, both, or neither.

An ExternalIntegration contains the configuration for connecting to a third-party API.  Commonly used third-party APIs include the metadata wrangler, DataSources that require protocols, authentication services, storage services, and search providers.

