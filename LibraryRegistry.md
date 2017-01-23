The library registry is a new Library Simplified product, the equal of the circulation manager or metadata wrangler. A library registry is strongly associated with a mobile application and with Adobe's VendorID specification.

NYPL will operate a library registry containing all libraries that participate in the SimplyE mobile app. If you want to run your own mobile app, you'll need to set up your own library registry, or create the equivalent in static OPDS feeds.

# Overview

The SimplyE library registry will start out as a static OPDS navigation feed listing a small number of participating libraries (NYPL, Brooklyn, Instant Classics and Open Ebooks). By the time the first Connecticut consortium launches, it will be a web application that guides patrons to the library or libraries that serve them.

This document explains what information a library or consortium needs to join the library registry. For implementation details see the [[LibraryRegistryDesign|library registry design document]].

# Joining the registry

The first step is to set up an OPDS server for your library. For most libraries this will be a Library Simplified circulation manager.

The second step is to gather the information below and send it to leonardrichardson@nypl.org (TODO: make this a special purpose address). We will help you fill in the gaps and verify that your OPDS server works before adding your library to the registry.

There is a lot of information here, but this is the information we need to make sure your OPDS server works and your patrons can locate your library from within SimplyE.

# For all libraries

All this information is required for all libraries unless marked "(optional)".

## Circulation URL

The root URL of your OPDS server. If you have your own circulation manager, this will be the URL to that circulation manager. If you share a circulation manager with other libraries, this will be your library-specific URL on that circulation manager. If you have created custom OPDS software or static OPDS feeds, this will be the root URL for that piece of software or those feeds.

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

## Sample credential

Before adding your OPDS server to the registry, we need to test it to make sure it works. Giving us a sample credential will let us verify that we can connect to your server with the SimplyE app and borrow a book.

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

Say which area you serve, and be specific. ("Serving Springfield, IL", not "Serving Springfield" or "Serving the Springfield area". "Orange County, CA", not "Orange County".). Remember the goal here is to help someone who's looking at a list of search results figure out which library is theirs.

## Cities covered

Local libraries generally serve a single city or town. List all the town names you serve, each with its state. If you serve a large city, you may also want to list the names of neighborhoods. If you serve an area larger than a city, you may also want to include county names or the names of other political divisions.

## Postal codes covered (semi-optional)

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

If all the libraries in your consortium have the same circulation URL, you can also add an entry for the consortium itself that points to that URL.

