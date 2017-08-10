# Home page

Currently the metadata wrangler home page gives a 404. Proposal is to change it to an OPDS 2 feed (`application/opds+json`) that links to the services the wrangler offers.

Example:

```
{
 "navigation" : [
   { "rel": "http://opds-spec.org/sort/new",
     "href": "/collection/updates",
     "type": "application/atom+xml;profile=opds-catalog",
     "title": "Recent changes to your tracked collection"
   },
 ]

 "links" : [
   { "rel": "register",
     "href": "/register",
     "type": "application/opds+json;profile=https://librarysimplified.org/rel/profile/metadata-service",
     "title": "Register your OPDS server with this metadata service"
   },
   { "rel": "http://librarysimplified.org/rel/metadata/lookup",
     "href": "/lookup{?urn}",
     "type": "application/atom+xml;profile=opds-catalog",
     "template": true,
     "title": "Look up metadata about one or more specific items"
   },
   { "rel": "http://librarysimplified.org/rel/metadata/collection-add",
     "href": "/collection/add{?urn}",
     "template": true,
     "title": "Add items to your collection."
   },
   { "rel": "http://librarysimplified.org/rel/metadata/collection-remove",
     "href": "/collection/remove{?urn}",
     "template": true,
     "title": "Remove items from your collection."
   },
   { "rel": "http://librarysimplified.org/rel/metadata/resolve-name",
     "href": "/canonical-author-name{?urn,display_name}",
     "template": true,
     "type": "text/plain",
     "title": "Look up an author's canonical sort name"
   },
 ]
}
```

Almost all of these features are already implemented, but their protocols need formal definitions and possibly some small changes.

# Collection registration

The core feature of the metadata wrangler is the delta feed, which tells you the latest information about every book in your collection. To get a delta feed, you need to register your collection with the metadata wrangler, and establish a shared secret you can use to authenticate yourself.

# Collection management

Once you have registered a collection, you can use the `http://opds-spec.org/sort/new` link to get the delta feed. But there won't be anything in the delta feed until the metadata wrangler knows which titles are in your collection.

When you send an authenticated request to the Metadata Lookup Service, all the IDs you mention are implicitly added to your collection.

You can also use the http://librarysimplified.org/rel/metadata/collection-add and http://librarysimplified.org/rel/metadata/collection-remove links, to explicitly add and remove IDs from your collection.

# Metadata Lookup Service

This is a public service (at least for now) which lets you look up information about any supported ID (probably an ISBN) and get back an OPDS feed full of information.

# Name Resolution Service

This is a public service (at least for now) which takes an author name as you might see it on the front of a book, plus an optional identifier (identifying the book in question). It tries to return the author name as it would appear in a card catalog.