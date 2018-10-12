Library Simplified Server-side Data Model Overview



This data model is common between the circulation manager and the metadata wrangler, although some pieces are exclusively used by one component or the other. For example, only the circulation manager has lanes or patrons, and only the metadata wrangler has integration clients. The library registry component has a separate data model which is similar but much simpler.

Although the data model is very complex, it can be split up into six relatively simple systems:

* Bibliographic metadata
* 
*
*
*
*

For the sake of simplicity, this document will talk about "books", but the rules are the same for audiobooks and other forms of content.

# Bibliographic metadata

Bibliographic information is information _about_ books as opposed to the books themselves. A book's title, its cover image, and its ISBN are all bibliographic information--the text of the book is not. Bibliographic information flows into the circulation manager and metadata wrangler from a variety of sources, mainly OPDS feeds and proprietary APIs. We keep track of all this information and where it came from, and when necessary we weigh it, sort it, and boil it down into a small amount of information that can be used by other parts of the system.

## `Identifier`

An `Identifier` provides a way to uniquely refer to a particular book. Common types of `Identifier` include ISBNs and proprietary IDs such as Overdrive or Bibliotheca IDs.

An `Identifier` may

* Serve as the primary identifier for multiple Editions
* Serve as the identifier for many LicensePools, through Collection
* Have many LicensePoolDeliveryMechanisms, through LicensePool
* Have many HyperLinks connecting it to external Resources, such as cover images or descriptions.
* Have many Measurements of the various numerically-quantifiable qualities (such as popularity, rating, page count, number of published editions, etc.) of the book to which it refers
* Have many Classifications, each of which categorizes the Identifier by assigning it to a different Subject.  A Subject designates the general categories that the book referenced by the Identifier falls under, such as target audience or age range, genre, or format.
* Be associated with one Work, through Edition
* Participate in many Equivalencies.  An Equivalency is an assertion made by a particular DataSource, according to which two different Identifiers are in fact referring to the same book, and are therefore equivalent.  

## `Edition`

An `Edition` is a collection of information about a book from a particular source. If different data sources provide conflicting information about a book, each data source is given its own `Edition` and we sort it out later.

An `Edition`:

* Has one Identifier  
* May be the presentation edition*i.e. the edition visible to library patrons*for a specific Work.
* Contains metadata*including title, series, author, language, publisher, medium, and cover image*for the Work to which it belongs
*May have one or more Contributors, through Contributions.  A Contributor is anyone who has contributed to the book in some capacity.  Some common examples of Contributors include authors, editors, translators, illustrators, and narrators.

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

