# Library Registry

The library registry is a new Library Simplified product, the equal of the circulation manager or metadata wrangler. A library registry is strongly associated with a mobile application and with Adobe's VendorID specification.

NYPL will operate a library registry containing all libraries that participate in the SimplyE mobile app. If you want to run your own mobile app, you'll need to set up your own library registry, or create the equivalent in static OPDS feeds.

# Overview

The library registry has two main features:

1. Provide OPDS navigation feeds that a user of your mobile app can use to find their local library.
2. (Optional) Implement the Adobe Authentication Web Service Specification so you can hand out Adobe IDs to users of your mobile application.

Note that #2 is currently the responsibility of the circulation manager. We're moving it to the library registry because very few organizations need this feature, and the ones that do will also probably need a library registry.

# Information per library

To join a registry, a library must provide the following:

- Display Name - A unique name shown to patrons, e.g. “Brooklyn Public Library”.
- Logo - Shown in the app when the library is selected. See below.
- Color - one of a fixed set of values to be used as the text color in the app when the library is selected
- Circulation Manager URL
- Enabled/disabled flag - disabled libraries will be temporarily hidden from the registry’s OPDS feeds
- Coverage area - Either a single lat/long representing the approximate center of the library's coverage area, or a polygon representing the entire coverage area. This lets the registry guide patrons to libraries near their current location. 
- Alternate coverage criteria, if any (see below)
- Vendor ID information (see below)

### Logo

Requirements for size, aspect ratio, transparency and palette are TBD. The image should be mirrored somewhere by the registry provider so it won’t break if the original link stops working.

### Alternate coverage criteria

Most registry entries represent public libraries who serve everyone in a small geographic area. On its own, "coverage area" handles these collections. But some collections are different.  Instant Classics and the Internet Archive's Open Library serve everyone with Internet access. Open Ebooks and NLS BARD cover the entire United States, but serve only subsets of the US population. It is not practical to specify in machine-readable form the requirements for every library of this type. We need a human-readable description of who qualifies for access for each such collection, and we should be able to create some basic groupings: "universal", "educational", "accessibility", "commercial", etc.

### Vendor ID information

Library Simplified circulation managers may be configured with a short library name and a shared secret. A circulation manager thus configured will start giving out tokens that may be used to activate a device with Adobe, using an Adobe ID provided by the library registry. For this to work, the library registry must know each library's short library name and shared secret, and the short library name must be unique across all that registry's libraries.

If a library does not authenticate patrons, or does not offer any books that are encrypted with Adobe DRM, there is no need to configure the Vendor ID information.

## Adding a library to the registry
For the initial version of the registry, someone from the simplified team will add libraries to the database manually.

Later, the registry will have a form for a library to fill out when its circulation manager is ready. This will add the library to the database. It might be disabled until someone from the simplified team reviews and enables it. We’ll need to determine what this process should be, and do as much validation as possible automatically when the form is submitted. 


## OPDS Feeds
The library registry provides a set of OPDS navigation feeds, where each library entry links to a library’s circulation manager URL, and provides display information needed by the clients.

Example library entry:
```
<entry>
  <id>1</id>
  <title>New York Public Library</title>
  <content>New York Public Library</content>
  <updated_at>2017-01-01T00:00:00Z</updated_at>
  <link rel="http://opds-spec.org/catalog" href="https://circulation.librarysimplified.org“ title=“New York Public Library“/>
  <link rel="alternate" href="/entry/1" type="application/atom+xml;type=entry"/>
  <category term=“http://registry.librarysimplified.org/logos/1.png” scheme="http://librarysimplified.org/libraryregistry/logo” label=“Logo”/>
  <category term=“#E32B31” scheme="http://librarysimplified.org/libraryregistry/color” label=“Color”/>
</entry>
```

The registry will have a top-level navigation feed that links to additional navigation feeds:
- Feed of all libraries sorted by name
- Hierarchy of feeds by location
- Nearby libraries - this link will detect your location based on IP and redirect you to an appropriate feed in the location hierarchy. The mobile client could also send it a location from its GPS to be used instead of IP.
- Search by library name
- Search by location

For the initial version, there can be a single flat navigation feed with an entry for every library.