The library registry is a new Library Simplified product, the equal of the circulation manager or metadata wrangler. A library registry is strongly associated with a mobile application and with Adobe's VendorID specification.

NYPL will operate a library registry containing all libraries that participate in the SimplyE mobile app. If you want to run your own mobile app, you'll need to set up your own library registry, or create the equivalent in static OPDS feeds.

# Overview

The library registry has two main features:

1. Provide OPDS navigation feeds that a user of your mobile app can use to find the circulation managers that participate in your app.
2. (Optional) Implement the Adobe Authentication Web Service Specification so you can hand out Adobe IDs to users of your mobile application.

Note that #2 is currently the responsibility of the circulation manager. We're moving it to the library registry because very few organizations need this feature, and the ones that do will also probably need a library registry.

When it comes to helping people find circulation managers, we will start out by supporting these four use cases:

1. Find sources of books that serve a given postal code.
2. Find sources of books that serve a given town or city.
3. Find sources of books based on geolocation of incoming IP address. (By estimating city from IP address and searching for a city.)
4. Browse sources of books that serve the whole world or an entire country.

# Use cases

The Cyrenius H. Booth Library serves the town of Newtown, Connecticut. This library is part of the Bibliomation consortium, which provides their ebook service. Someone from Newtown who comes into SimplyE and looks for 'their' library is likely to search for "Newtown", not "Cyrenius" or "Bibliomation". If they choose "find libraries near me" and the only result is called "Bibliomation" they are likely think their library is not on SimplyE.

The 53rd Street Library is a branch of the New York Public Library. Someone who thinks of that branch as 'their' library will probably go into SimplyE and search for "NYPL" to find "their" library. They're probably not going to search for "53rd Street" or "New York Public Library". If they choose "find libraries near me" they will expect to see "New York Public Library", not "53rd street", "Columbus Library", "Mid-Manhattan Library", etc.

The Internet Archive's Open Library is open to everyone with Internet access, but relatively few people will open up SimplyE looking for it.

The Open Ebooks collection is open to a lot of the schoolchildren in the United States, including some who are too young to navigate a complex library selection screen. Open Ebooks can be thought of as a library that covers the entire United States but which does not serve everyone in that geographic region.

# Information per library

To join a registry, a library must provide the following:

- Display Name - A unique name shown to patrons, e.g. “Brooklyn Public Library”.
- Description - A short description of the purpose of the library and the population it serves.
- Logo - Shown in the app when the library is selected. See below.
- Color - one of a fixed set of values to be used as the text color in the app when the library is selected
- Circulation Manager URL
- Enabled/disabled flag - disabled libraries will be temporarily hidden from the registry’s OPDS feeds
- Coverage area - Either a single lat/long representing the approximate center of the library's coverage area, or a polygon representing the entire coverage area. This lets the registry guide patrons to libraries near their current location. 
- Alternate coverage criteria, if any (see below)
- Short name for use in Adobe ID generation (see below)
- Shared secret for use in Adobe ID generation (see below)

### Logo

Requirements for size, aspect ratio, transparency and palette are TBD. The image should be mirrored somewhere by the registry provider so it won’t break if the original link stops working.

### Alternate coverage criteria

Most registry entries represent public libraries who serve everyone in a small geographic area. On its own, "coverage area" handles these collections. But some collections are different.

* Instant Classics and the Internet Archive's Open Library serve everyone with Internet access.
* Open Ebooks and NLS BARD cover the entire United States, but serve only subsets of the US population. 

It is not practical to specify in machine-readable form the requirements for every library that doesn't fit the "everyone in a small area" mold. Instead, we need a human-readable description of who qualifies for access for each such collection. We should be able to create some basic groupings: "universal", "educational", "accessibility", "commercial", etc.

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

# Configuration

* Database - to hold library information and Adobe IDs.
* S3 bucket - to hold library icons

# Other designs

We know of two existing library registries that do something similar to what we want to do.

## Feedbooks Library Search

URL: [[http://library-search.feedbooks.net/|http://library-search.feedbooks.net/]]

Feedbooks Library Search is based on OPDS, as our solution will be. The hierarchy of OPDS feeds starts with a list of countries. Choose a country ("United States") and you get a list of regions ("Florida"). Choose a region and you get a list of libraries. This can contain a very large number of libraries: 1343 in California.

You can perform a search using the normal OPDS search capability (by following the link with `rel="search"`).

* [[A search for "Bakersfield"|http://library-search.feedbooks.net/search.atom?query=bakersfield]] finds libraries in or near Bakersfield, CA, even if they don't have "Bakersfield" in the name.
* [[A search for "state university"|http://library-search.feedbooks.net/search.atom?query=state+university]] finds university libraries from around the US.

The link with `rel="recommended"` links to [[http://library-search.feedbooks.net/featured.atom|http://library-search.feedbooks.net/featured.atom]], a list of "Nearby libraries" based on geolocation of the requester's IP address. This is not particularly accurate: from NYPL SASB it recommended a set of libraries in Connecticut, even though NYPL shows up as the first result when you search for "New York".

Postal code search works inconsistently. Search for a ZIP code in Queens turns up one result. Search for a ZIP code in Irvine turns up nothing, even though a search for "Irvine" turns up several results.

## Overdrive "Add a library"

We're not privy to what happens behind the scenes when you choose to 'add a library' to your Overdrive app, but it gives us a view into what the interface might look like. It all starts from two options: search for a library "by name, city or postal code", or browse the list of libraries.

### Search 

Performing a search yields a flat list of libraries, each with its city, state and country listed. Tap a library and you'll see its street address and be able to add it to your Overdrive app. Branches of a multi-branch library show up separately in the list, although they all correspond to the same Overdrive collection. This would be useful if you needed to know the address of a nearby branch library, but it clutters the results. 

However, sometimes the cluttered results are helpful. The libraries in Irvine, CA are managed by the county library. Searching for "Irvine" does not pick up the Orange County Public Library. It only picks up the two branches of that library located in Irvine.

We can learn a lot by trying common searches against Overdrive's list of libraries. Searching for "NYPL" or "Bakersfield" gives good results. Searching for "queens" does not give good results. Searching for "queens ny" turns up nothing.

I searched for my Queens ZIP code and was given a large number of nearby branches of the Queens Library. The first result that did not come from Queens was a library in Honduras whose Honduran postal code was similar to my ZIP code. Then came a number of other Queens Library branches, then the Birch Wathen Lenox School, the Allen-Stevenson School, a number of other school libraries, and finally a NYPL branch: the 67th Street Branch. I would expect a postal code search to turn up the Queens Library, NYPL, the Brooklyn library, and the libraries for the various nearby schools.

Searching for an Irvine ZIP code gave much better results. The top nearby results were all branches of  the Orange County Public Library. But just returning a single result, "Orange County Public Library", would have done the same job.

### Browse

If you choose to browse for libraries, you're given a list of countries (with flags) with the United States at the top. Click on United States and you're given a list of states (with flags). Then you get a list of every single library in California with an Overdrive account, in alphabetical order.

Click on Belgium instead, and you get a single link to click on: "Brussels". Click on "Brussels" and you get a single library.

Click on Canada and you get a list of provinces, without flags. Nunavut is not included in the list, presumably because there are no libraries there with an Overdrive account.

It's common for a country to have itself as a subsection. This happens with Brazil, China, Qatar, Saudi Arabia and Switzerland, to name just a few of the countries I checked. This could be a place to put nationwide libraries, but in practice that doesn't seem to be how it works. (The national library of Qatar is under Qatar/Doha, not Qatar/Qatar.)

### Conclusion

Browsing the library list is a use case in search of a user. A list of all the libraries in California is useless, especially if you can't filter or sort it. A list of all the Overdrive libraries in Belgium is useful, because there's only one such library--but searching for "Belgium" or "Brussels" should also give you that library.

Searching is also a mess, but it's not obvious how to improve it in a way that benefits all the use cases above.