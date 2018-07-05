The Library Simplified project consists of a small number of
architectural components, joined together by a single protocol --
OPDS. This document lists the main components of the system and shows
how they fit together.

# Server components

## The circulation manager

The [circulation
manager](https://github.com/NYPL-Simplified/circulation) mediates
between a library's patrons, the library's ILS, and the many external
vendors from which libraries license ebooks and audiobooks.

The circulation manager provides patrons with a single consolidated
catalog which may contain electronic content from any number of
vendors. When the patron expresses interest in a particular title, the
circulation manager validates their credentials with the library's
ILS. Once the credentials are validated, the circulation manager
contacts the external vendor and negotiates the loan and delivery of
the title to the patron.

The circulation manager hides the differences between library ILS
systems and content vendors. A single circulation manager may manage
the circulation for many libraries -- this is most useful for library
consortia that have a shared collection.

We expect the Library Simplified ecosystem to contain hundreds or
thousands of deployed circulation managers, each managing the
collection for a relatively small (1-50) number of libraries.

We also expect the Library Simplified ecosystem to contain many
software installations which authenticate patrons, negotiate loans,
and deliver books, but which are not based on the circulation manager
code base. An example is [the
Internet Archive's book server](https://bookserver.archive.org/catalog/). The circulation manager _software_ is
less important than the architectural _role_ it fulfills.

## The library registry

The [library
registry](https://github.com/NYPL-Simplified/library_registry) is a
discovery mechanism for matching human beings to the libraries that
serve them. People are matched to libraries primarily by geographic
proximity, but they may also be matched to libraries that serve
specific audiences, such as students, or people with print
disabilities.

A properly configured circulation manager can automatically register
itself with a library registry.

We expect the Library Simplified ecosystem to contain only one library
registry. We expect every library in the ecosystem to be registered
with that registry. If there are multiple library registries, it
becomes much more difficult for patrons to find 'their' libraries, since they would first have to find 'their' library registry. (If this happens, we could mitigate it by creating a second discovery protocol and running searches on multiple registries.)

## The metadata wrangler

The [metadata
wrangler](https://github.com/NYPL-Simplified/metadata-wrangler) exists
to find the best available bibliographic metadata for every book in
the world, and to share this metadata with circulation managers.

A properly configured circulation manager can be registered with a
metadata wrangler. When a title is added to the circulation manager's
collection, it will notify the metadata wrangler, and 'subscribe' to
metadata updates for that title. Whenever the metadata wrangler finds
improved metadata for that title, the circulation manager will hear
about it and start using the new metadata.

We expect the Library Simplified ecosystem to contain a relatively
small number of metadata wranglers, since bibliographic information
about books can be shared across libraries. However, unlike with the
library registry, nothing is lost if individual organizations decide
to set up their own metadata wranglers -- it's just inefficient.

# Client components

A patron of a library uses a client component to communicate with
their library's circulation manager. Client components are generally
designed for the use of library patrons. Patrons use client components to browse a library's catalog, to borrow books and audiobooks, and to read or listen to the books and audiobooks.

A client component may also communicate with the library registry.
This would allow patrons to find their library from within the client,
rather than having to locate their library first (e.g. by a web search) and finding out if that
particular library has a client available.

## Mobile applications

The most commonly used client is SimplyE, a mobile application that communicateswith the library registry and with circulation managers. There is an [iOS client](https://github.com/NYPL-Simplified/Simplified-iOS) and an [Android client](https://github.com/NYPL-Simplified/Simplified-Android), both of which are called "SimplyE".

SimplyE is the preferred client for most purposes, because it's able
to negotiate DRM licenses for ebooks and audiobooks. Someone using
another client will only be able to consume media that is not
encrypted with DRM.

## Web applications 

The [patron-facing web client](https://github.com/NYPL-Simplified/circulation-patron-web) is a web application that offers much of the functionality of SimplyE within a web browser. Patrons using the web client can browse a catalog, authenticate with their library's ILS, see their current loans, and borrow books and audiobooks. However, the patron-facing web client cannot negotiate DRM licenses for ebooks or audiobooks, so this client can only retrieve titles that are not encrypted with DRM.

Like the circulation manager, the patron-facing web client must be installed for a specific library. The patron-facing web client does not communicate with the library registry and does not have an interface to locate "your library".

## General OPDS clients

Since circulation managers are OPDS servers (see below), to some
extent you can use any OPDS client to access a circulation
manager. This includes mobile clients such as [Aldiko](http://www.aldiko.com/).

In practical terms, the most useful general OPDS client is the [OPDS browser demo](http://opds-browser-demo.herokuapp.com/). By putting in the URL to any OPDS server, you can browse that server and conduct transactions from your web browser. [Here's an example using the Internet Archive's book server.](http://opds-browser-demo.herokuapp.com/collection/https%3A%2F%2Fbookserver.archive.org%2Fcatalog%2F/)

# OPDS: the magic arrow

Here's a simple architectural diagram showing how the user of a client
(a library patron) gets a list of libraries from the library registry
and a catalog of books from a circulation manager. It shows how
bibliographic metadata flows from the metadata wrangler to the
circulation manager, and how ebooks might flow from a distributor
through the circulation manager to the patron.

```                 
+--------+          +------------------+
| Client |<--OPDS---| Library registry |
+--------+          +------------------+  
   ^            
   |
   |         +---------------------+          +-------------------+
   +--OPDS---| Circulation manager |<--OPDS---| Metadata wrangler |
             +---------------------+          +-------------------+
                       ^ 
                       |                      +-------------------+
                       +---OPDS---------------| Ebook distributor |
                                              +-------------------+
```

Every communication in this diagram uses the same protocol: OPDS. OPDS is basically a general way of talking about books. OPDS can represent anything to do with  books. This includes bibliographic metadata (like that produced by the metadata wrangler), catalogs (like those generated by the circulation manager, or links to the books themselves (like those provided by distributors).

There are two different versions of OPDS: [OPDS 1.2](https://github.com/opds-community/opds-revision/blob/master/opds-1.2.md), which is an XML-based format and the forthcoming [OPDS 2.0](https://github.com/opds-community/opds-revision/blob/master/opds-2.0.md), which is a JSON-based format. Both versions of OPDS serve the same purpose -- to convey information about books.

OPDS doesn't cover absolutely everything that Library Simplified needs
to do. We've invented a lot of extensions to OPDS, as well as totally
separate protocols like the [OPDS Directory Registration
Protocol](https://github.com/NYPL-Simplified/Simplified/wiki/OPDS-Directory-Registration-Protocol). But
the core design principle of Library Simplified is that the components
should communicate using OPDS as much as possible.

