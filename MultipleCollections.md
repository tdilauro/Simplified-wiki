A collection is a set of titles combined with the credentials necessary to distribute those titles. Some examples:

* Brooklyn Public Library's Overdrive collection
* The Bibliomation Overdrive collection
* The state of Connecticut's One Click Digital collection
* NYPL's Bibliotheca collection
* NYPL's Axis 360 collection
* The "Instant Classics" public domain collection, available from the content server.
* The "Recovering the Classics" public domain collection, available from the content server.

One library may have more than one collection. In fact, one library may have more than one collection from the same source! (Think of a library that participates in a statewide Overdrive collection but supplements it with its own Overdrive collection.)

Currently, books come from a data source ("Overdrive"). This proposal introduces the notion of a 'collection' ("statewide Overdrive collection") as the ultimate source of books. A data source is recast as a place you _can_ get books from if you have the right credentials.

# Database schema changes

Rename the existing `collections` table to something else. That table tracks the relationship between a circulation manager and the metadata wrangler.

Create a new `collections` table that looks like this:

```
collections
 id (primary key)
 name (unique)
 data_source_id
 external_account_id
 username
 password
 url
```

This table tracks the most common configuration settings necessary to set up a collection: the ID of the collection's account on the external service, generally called a "library ID", the credentials (username/password or key/secret) necessary to gain access to the collection, and potentially a URL (necessary for OPDS or ODL-based integrations).

Each collection is also associated with a data source (`data_source_id`). The data source controls which API is actually used to get books from this collection. This also serves as a drop-in replacement for `licensepools.data_source_id`. (It may be necessary to use a special `type` field to determine which API to use, I'm not sure.)

Any left over configuration settings will go into the `collectionsettings` table:

```
collectionsettings
 collectionsetting_id
 collection_id (foreign key for collections.id)
 key
 value
unique (collection_id, key)
```

The `licensepools.data_source_id` field will be removed and replaced with a `licensepools.collection_id`. 

## Database migration

Script #1 will create a collection for each distinct `data_source_id` in `licensepools`. This can probably be done in SQL. Each collection will be named after its data source.

Script #2 will be a Python script which configures the new collections with data from the site configuration. After this point the collection configuration can be removed from the site configuration JSON.

In theory this should work for all three components (content, circulation, metadata).

# Third-party API changes

Instead of getting configuration from the JSON configuration via the Configuration object, all APIs that offer access to content (e.g. OverdriveAPI, OneClickAPI) will get their configuration from a Collection object passed in to the constructor.

Each API will derive from an abstract base class called `CollectionAPI` that lays out the methods any collection API must support. One such method will be a `self_test` method which verifies that the credentials provided are working and that the API is up and running. The `self_test` method needs to return a data structure that includes status and timing information for various common activities.

# Script changes

Scripts that deal with a collection API, like the `overdrive_monitor_*` scripts, will need to deal with each collection separately. First off, they should take a `collection` command-line argument so you can run them on a specific collection that's having problems. When run without this argument, they need to operate on every collection at once. This might mean spawning multiple threads to handle all collections simultaneously (which could reveal previously hidden bugs) or it might mean iterating over the list of collections, handling one at a time.

# Web app changes

## Identifying books

A web app URL that identifies a specific book looks like this:

```
https://circulation.librarysimplified.org/works/3M/Bibliotheca%20ID/{id}/{action}
```

"3M" here is the name of the data source. This will be replaced with the name of the collection.

In all currently deployed circulation managers, the name of the collection will be the same as the name of the data source, so old links will continue to work.

While we're at it, if there is a metadata wrangler configured we should do a self-test of the metadata wrangler, even though that's not a collection.

## Service status

The `service_status` controller should be changed to iterate over all the collections and call the self test on each one. It should then present the status and timing information returned by the self test.

This controller should be made part of the administrative interface, unless that would stop it from being automatically accessible and easily parsable by a monitoring tool.

# Admin interface

We need interfaces for doing the following:

* List collections
* Create a new collection (from a preset list of types of collections)
* Edit a collection
* Run the self-test on a collection

Removing a collection should only be possible if there are no license pools associated with the collection.

When you load the list of collections, it should (in the background) trigger the self-test for each one and display the results as they come back.