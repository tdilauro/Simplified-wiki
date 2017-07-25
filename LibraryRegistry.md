# Overview

A library that sets up a Library Simplified circulation manager can use the administrative interface to set information about the library that is useful in helping patrons find the library: for instance, the library's name and the cities it serves. When the library is ready to go live, it syncs this information with its library registry. The job of the library registry is to consolidate information about many libraries and to make it searchable, so that people can find the library or libraries that serve them.

NYPL runs a library registry, and all libraries that participate in the SimplyE mobile app must sync their information with this registry. If you want to run your own mobile app along the lines of SimplyE, you'll need to set up your own library registry, or create the equivalent in static OPDS feeds.

The [original design document for the library registry](LibraryRegistryDesign) is still available.

# Syncing

Synchronization between the circulation manager (or equivalent) and the library registry is done through the Authentication for OPDS document. All information needed by the library registry is defined in [Authentication for OPDS Extensions](Authentication-For-OPDS-Extensions).

To synchronize a library with the registry, the circulation manager finds the link with rel "register" in the registry's main feed, and sends a POST request with the "url" parameter set to the url of the library's OPDS feed. The registry attempts to find the Authentication for OPDS document. The document may be in the body of the response if the feed requires authentication. Otherwise, the registry will look for a link with rel "http://opds-spec.org/auth/document" or rel "http://opds-spec.org/shelf".

## Sync response
In response to a sync operation, the library registry will provide the library's entry in the registry as an [OPDS 2.0](https://github.com/opds-community/opds-revision/blob/master/opds-2.0.md) Catalog, or a problem detail if the sync failed. 

### Permalink
A successful sync response will contain a permalink to the library's registry entry with rel "self" under "links". The library can check the permalink to determine if the library is live on the registry yet.
```
{
  "metadata": {
    "title": "The New York Public Library"
  },
  
  "links": [
    {"rel": "self", "href": "https://registry.librarysimplified.org/libraries/1", "type": "application/opds+json"}
  ],
  
  "navigation": [
    {
      "href": "https://circulation.librarysimplified.org/groups", 
      "title": "The New York Public Library", 
      "type": "application/atom+xml;profile=opds-catalog;kind=acquisition",
    }
  ]
}
```

### Shared secret
If the library's Authentication for OPDS document contains a public key, the library registry will also provide an encrypted shared secret and short library name, for use in signing [Short Client Tokens](Short-Client-Token) that the registry can verify. This gives the library registry a way to validate that library's patrons, without needing access to the library's ILS. The registry will generate these values if they don't exist yet. The shared secret will be encrypted with the library's public key using the RSAES-OAEP scheme, and then base 64 encoded.
```
{
  "metadata": {
    "title": "The New York Public Library",
    "shared_secret": "jZdwjQPOO+5oKu797iCstMvVr6/+F6PIr0S4nmV7MlO",
    "short_name": "abcd"
  },
  
  "links": [
    {"rel": "self", "href": "https://registry.librarysimplified.org/libraries/1", "type": "application/opds+json"}
  ],
  
  "navigation": [
    {
      "href": "https://circulation.librarysimplified.org/groups", 
      "title": "The New York Public Library", 
      "type": "application/atom+xml;profile=opds-catalog;kind=acquisition",
    }
  ]
}
```

# Work to be done

- Everything in [Connecting the Circulation Manager to the Library Registry](CirculationManagerToLibraryRegistry)
- Set up a system for [making synced libraries 'live'](https://github.com/NYPL-Simplified/library_registry/issues/25) on the registry.