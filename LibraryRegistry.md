The library registry is a new Library Simplified product, the equal of the circulation manager or metadata wrangler. A library registry is strongly associated with a mobile application and with Adobe's VendorID specification.

NYPL will operate a library registry containing all libraries that participate in the SimplyE mobile app. If you want to run your own mobile app, you'll need to set up your own library registry, or create the equivalent in static OPDS feeds.

# Overview

The library registry has two main features:

1. Provide OPDS navigation feeds that a user of your mobile app can use to find the circulation managers that participate in your app.
2. (Optional) Implement the Adobe Authentication Web Service Specification so you can hand out Adobe IDs to users of your mobile application.

Note that #2 is currently the responsibility of the circulation manager. We're moving it to the library registry because very few organizations need this feature, and the ones that do will also probably need a library registry.

The SimplyE library registry will start out as a static OPDS navigation feed listing a small number of participating libraries (NYPL, Brooklyn, Instant Classics and Open Ebooks). By the time the first Connecticut consortium launches, it will be a web application that acts as an Adobe ID server and supports these four use cases:

1. Find libraries that serve a given postal code.
2. Find libraries that serve a given town or city.
3. Find nearby libraries based on geolocation of incoming IP address.
4. Browse libraries that serve the whole world or an entire country.

Unlike Feedbooks and Overdrive, we do not plan to offer the ability to browse all participating libraries in a state or country.

# Use cases

* The Cyrenius H. Booth Library serves the town of Newtown, Connecticut. This library is part of the Bibliomation consortium, which provides their ebook service. Someone from Newtown who comes into SimplyE and looks for 'their' library is likely to search for "Newtown", not "Cyrenius" or "Bibliomation". If they choose "find libraries near me" and the only result is called "Bibliomation" they are likely think their library is not on SimplyE.

* The 53rd Street Library is a branch of the New York Public Library. Someone who thinks of that branch as 'their' library will probably go into SimplyE and search for "NYPL" to find "their" library. They're probably not going to search for "53rd Street" or "New York Public Library". If they choose "find libraries near me" they will expect to see "New York Public Library", not "53rd street", "Columbus Library", "Mid-Manhattan Library", etc.

* The Internet Archive's Open Library is open to everyone with Internet access, but relatively few people will open up SimplyE looking for it.

* The Open Ebooks collection is open to a lot of the schoolchildren in the United States, including some who are too young to navigate a complex library selection screen. Open Ebooks can be thought of as a library that covers the entire United States but which does not serve everyone in that geographic region.

# For all libraries

All this information is required for all libraries unless marked "(optional)".

## Circulation URL

The OPDS root for your library. If you have your own circulation manager, this will be the URL to that circulation manager. If you share a circulation manager with other libraries, this will be your library-specific URL on that circulation manager. If you have created custom OPDS software or static OPDS feeds, this will be the root URL for that piece of software or those feeds.

## Display name

The name of your library, as it should be shown to patrons. e.g. "Brooklyn Public Library"

## Description

A short human-readable explanation of who the library serves. This shows up in search results and helps a patron figure out which "Springfield Public Library" is the one they want, or whether they can get books from this particular "Library for the Blind".

For guidance on what the description should look like, see your type of library below.

Use the language you expect your patrons to read. (TODO: we need a localization strategy for bilingual libraries.)

In general, this is not a great place for feel-good phrases like "serving with pride". People are trying to find a library that serves _them_ in particular.

## Adobe ID information (optional)

This is marked as optional, but if your library controls access to any books that are encrypted with Adobe DRM, you really should provide it. If you have any books from Overdrive, Bibliotheca, or Axis 360, you need this. Providing it when it's not necessary does no harm.

There are two pieces of information: 

### Short name

A string eight characters or less and must uniquely identify your library. For libraries in the United States, it should start with the two-letter state abbreviation. The SimplyE team will help you come up with a unique name. This name is never displayed to patrons.

### Shared secret

The SimplyE team will generate this string and share it with you. It needs to go into your circulation manager's configuration.

## Logo (optional)

A small graphic representing your library. This is shown in SimplyE when your library is selected. It's also shown when your library shows up in lists. 

Requirements for size, aspect ratio, transparency and palette are TBD.

## Color (optional)

One of a fixed set of values to be used as the core color in the app once the library is selected. If your library uses a distinctive color scheme, choose the color that best fits your scheme. By default, SimplyE uses a grey color scheme.

## Online signup link (optional) 

If you offer your patrons the opportunity to sign up for a library card online, and you want us to advertise this fact in the registry, send us the link to the signup form. To actually get this to work, you will also need to configure your circulation manager or Authentication for OPDS document with the online signup link. We don't store this link, we only need it for verification purposes so we can flip a yes/no switch.

## No registration required (optional)

Most libraries require patrons to register and authenticate to gain access, but some are open to everyone without requiring registration. If your library is like this, let us know and we'll explain this to prospective patrons.

## Alternate names (optional)

If your library is known by an abbreviation or a nickname, provide it so we can find your library when people search for it. (Note that we will probably also find other libraries with the same abbreviation!)

# For local libraries

A local library serves a specific geographic community. We need to understand the locations and boundaries of that community, so we can guess whether someone is likely to be in the library's service area. We also need to know the names and numbers associated with the community, so we can guide its residents to their local library.

## Description guidelines

Say which area you serve, and be specific. ("Serving Springfield, IL", not "Serving Springfield" or "Serving the Springfield area"). Remember the goal here is to help someone who's looking at a list of search results figure out which library is the right one.

## Cities covered

Local libraries generally serve a single city or town. List all the town names you serve, each with its state. If you serve a large city, you may also want to list the names of neighborhoods. If you serve an area larger than a city, you may also want to include county names or the names of other political divisions.

## Postal codes covered

Provide the ZIP codes (or other postal codes) where your patrons live. This will help people find your library by searching for their postal code. If we have the list of cities you serve, we can generate a corresponding list of ZIP codes for you, which should be pretty accurate.

## Branch locations (optional)

Provide the latitude and longitude of your branch library building(s). This is a simple alternative to a coverage shapefile which you can measure with a GPS-enabled smartphone.

## Coverage shapefile (optional)

If you have an ESRI shapefile for the area your library covers, provide it. Most of the time it's easier just to provide city names, ZIP codes, and branch locations.

# For national libraries

National libraries either serve everyone in the nation, or a non-geographic subset of your nation, such as the visually impaired.

## Nation

We need to know which country you serve so we can group you with other libraries in that country.

## Description guidelines

Again, explicitly state which nation you serve. 

If you serve everyone in your nation, say so. If there are other national-scale libraries, also explain what's in your collection, so that patrons can distinguish between your library and other national libraries.

If you serve a non-geographic subset of your nation, such as the visually impaired, explain who you serve and also mention what's in the collection. 

# For universal libraries

Universal libraries are open to everyone in the world.

## Description

Say that your library is open to everyone and explain what's in your collection.

# For library consortia

If you represent a group of local libraries who are all joining SimplyE, you have two choices. You can have a separate entry for each individual library, or you can have one entry for the consortium itself.

In general, we recommend a separate entry for each individual library, even if all the entries go to the same circulation URL. We're trying to help people locate their library, and most people think of their library in terms of the city they live in, not the consortium the library belongs to.

# Adding a library to the registry

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