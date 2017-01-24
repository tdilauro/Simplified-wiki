
The library registry has two main features:

1. Provide OPDS navigation feeds that a user of your mobile app can use to find the OPDS servers that participate in your app.
2. (Optional) Implement the Adobe Authentication Web Service Specification so you can hand out Adobe IDs to users of your mobile application.

Note that #2 is currently the responsibility of the circulation manager. We're moving it to the library registry because very few organizations need this feature, and the ones that do will also probably need a library registry.

# Use cases

1. Find libraries that serve a given postal code.
2. Find libraries that serve a given town or city.
3. Find nearby libraries based on geolocation of incoming IP address.
4. Browse libraries that serve the whole world or an entire country.

Unlike Feedbooks and Overdrive, we do not plan to offer the ability to browse all participating libraries in a state or country.

# Examples

* The Cyrenius H. Booth Library serves the town of Newtown, Connecticut. This library is part of the Bibliomation consortium, which provides their ebook service. Someone from Newtown who comes into SimplyE and looks for 'their' library is likely to search for "Newtown", not "Cyrenius" or "Bibliomation". If they choose "find libraries near me" and the only result is called "Bibliomation" they are likely think their library is not on SimplyE.

* The 53rd Street Library is a branch of the New York Public Library. Someone who thinks of that branch as 'their' library will probably go into SimplyE and search for "NYPL" to find "their" library. They're probably not going to search for "53rd Street" or "New York Public Library". If they choose "find libraries near me" they will expect to see "New York Public Library", not "53rd street", "Columbus Library", "Mid-Manhattan Library", etc.

* The Internet Archive's Open Library is open to everyone with Internet access, but relatively few people will open up SimplyE looking for it.

* The Open Ebooks collection is open to a lot of the schoolchildren in the United States, including some who are too young to navigate a complex library selection screen. Open Ebooks can be thought of as a library that covers the entire United States but which does not serve everyone in that geographic region.

# Administrative interface

For the initial version of the registry, the Library Simplified team will accept applications via email and add libraries to the database manually. 

Later, the registry will have a form for a library to fill out when its circulation manager is ready. This will add the library to the database. It might be disabled until someone from the simplified team reviews and enables it. We’ll need to determine what this process should be, and do as much validation as possible automatically when the form is submitted. 

# Resource design

The library registry provides a set of OPDS navigation feeds. Each `<entry>` represents a library, links to that library’s circulation URL, and provides display information needed by the clients.

## Top level

In the first version, the top-level resource is an OPDS navigation feed containing an `<entry>` for each in a preset list of libraries.

In the second version, the top-level resource is an OPDS navigation feed that links to an OpenSearch document (see below) and which contains an `<entry>` for a small number of nearby libraries. "Nearby" is determined using some as-yet-unknown geolocation technique. If geolocation takes too long, we will bail out and just show an empty feed.

It might be that we have to give certain libraries special treatment, (e.g. always put Open Ebooks on the front page so that kids won't have to navigate).

## OpenSearch document

The OpenSearch document explains how to run a search for your library.

```
<OpenSearchDescription>
 <ShortName>SimplyE Library Search</ShortName>
 <Description>Search for your library by name, city or postal code</Description>
 <InputEncoding>UTF-8</InputEncoding>
 <OutputEncoding>UTF-8</OutputEncoding>
 <Url type="application/atom+xml" template="https://catalog.librarysimplified.org/search.atom?query={searchTerms}"/>
</OpenSearchDescription>
```

It might be helpful to do another geolocation lookup and insert the name of the guessed city into a `<Query>` tag:

```
<Query role="example" searchTerms="Columbus"/>
```

More likely we should not bother. If someone needs to do a search it's probably because our attempt to find nearby libraries has failed them.

## Library entry

Example library entry:
```
<entry simplified:signup_policy="online">
  <id>1</id>
  <title>New York Public Library</title>
  <content>Serving the five boroughs of New York City, NY.</content>
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

# Database design

Hopefully we will be able to store library icons as binary objects in the database, and send them inline in the OPDS feeds using `data:` URIs. If not, we'll need to store them in an S3 bucket and keep track of the URLs.


## Libraries

Any institution that has a circulation URL is a library.

```
libraries
 id
 circulation_url
 display_name
 description
 logo or logo_url
 color
 patron_policy
 adobe_name
 adobe_secret
```

## Library Aliases

A library may have many aliases. An alias is an alternate name for the library organization itself.

```
libraryaliases
 id
 library_id
 name
```

"BPL" and "Bklyn" are aliases for the Brooklyn Public Library. "BPL" is also an alias for the Boston Public Library. "Brooklyn" is not an alias for the Brooklyn Public Library; it's a region (q.v.) the library serves.

## Regions

A library may serve many regions. Postal codes, cities, counties, states, provinces, and nations are all regions. Regions stack inside each other in various ways, but we are not interested in creating a perfectly accurate picture of political reality. We only need to do work if it helps people find their library. As such, although we keep track of a region's parent region, this is mainly for internal record-keeping.

```
regions
 id
 name
 abbreviated_name
 full_name
 parent_id
```

A library may serve many regions.

The `abbreviated_name` for the state of Ohio will be "OH".

The `full_name` for Detroit is "Detroit MI". The idea here is to have a place to store a common search term (the way you refer to a city when addressing a letter) without requiring an entry in `regionaliases` for every ordinary city in the United States.

## Region Aliases

A region may have many aliases. An alias is an alternate name for the region that we expect someone to use when searching.

```
regionaliases
 id
 region_id
 name
```

Aliases for New York City include "NYC" and "New York NY". (The automatically generated `full_name` of New York City will be "New York City NY", so we need an alias for the more common form.)

## Locations

A location represents a point or area on the globe.

```
locations
 id
 latitude
 longitude
 shapefile
```

If `shapefile` is provided as well as `latitude` and `longitude`, it's presumed that the latitude/longitude pair is a _representative_ point within the area described by the shapefile.

A library may be associated with multiple locations.

A region may be associated with multiple locations.

# Data population

We will create a region for each city, county, and state in the US, and associate each with a shapefile we got from TIGER. Each city and county will be given a `full_name` that appends its state. (e.g. "Orange County CA" and "Orange County FL").

We will create a region for each US ZIP code. The parent of a ZIP code region will be the city associated with that ZIP code in [[the Geonames dataset|http://download.geonames.org/export/zip/]]. The ZIP code region will be represented by the shapefile associated with the corresponding ZIP Code Tabulation Area (available from [[TIGER|https://www.census.gov/geo/maps-data/data/tiger-cart-boundary.html]]).

[[ZIP codes are not areas|http://www.georeference.org/doc/zip_codes_are_not_areas.htm]], but Zip Code Tabulation Areas are close enough for our purposes.

# Implementing the use cases

## Search by postal code

This is the simplest case, so we do it first. If there is a search query, we try to match it against `postalcodes.code`. This either gives us nothing (in which case we continue) or it gives us a single location.

Then we run a GIS query to find the libraries whose locations are nearest that location. These are the libraries we return.

## Estimate user's location based on IP address

In all other cases, we need to get a shapefile representing the user's approximate current location. We will either use this information directly to find a nearby library, or we will use it to sort search results by proximity.

## Find nearby libraries

If there is no search query, we need to get from the user's current location to nearby locations associated with libraries, either through region or postal code. This requires doing a GIS search against the `locations` table.

## Find matching locations

If there is a search query, we need to find locations that match the search. There are several ways of doing this.

### Search by library name

We run a search against `libraries.name` and `aliases.name`. This gives us a number of libraries, each with a number of associated locations. We sort the list by proximity to the user's current location.

### Search by city

We run a search against `regions.name` and `regions.abbreviated_name` and look up all regions that match and are associated with libraries. We sort the list by proximity to the user's current location.

We will need to handle queries like `paris, tx`. In this case the user is providing two region names and wants to find only regions that match both.

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