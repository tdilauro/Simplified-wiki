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

## Kicking off the registration process

Send a POST request to the `register` link with an
`application/x-www-form-urlencoded` form as payload. The form MUST
contain a single key-value pair, `url`, with the value being the URL to your metadata key document.

```
POST /opds-directory
Host: example.com
Content-Type: application/x-www-form-urlencoded

url=http://example.org/my-metadata-key
```

The directory will retrieve your metadata key document and try to create a shared secret. This shared secret covers _the entire domain_, and anyone who can put a file onto your domain can change it.

If the metadata key document is missing or the directory can't make sense of
it, you'll get a [problem detail
document](https://tools.ietf.org/html/rfc7807). If the registry is
able to acommodate your OPDS server, you'll get a registration
document with the media type
`application/opds+json;profile=https://librarysimplified.org/rel/profile/metadata-service`. This
document shows you what the directory was able to
derive from the information you gave it.

This document will contain a single key, `shared_secret`, containing the shared secret between you and the metadata wrangler.

## Metadata key document

This is an OPDS 2 document containing a single item of metadata,  `public_key`.

```
{ "metadata": {
    "public_key": { "type": "RSA", "value": "-----BEGIN RSA PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA6dM2j3D3ykxLfbZ4zm1o\nx6wzSvbUsL6rrdTQ+J/zduBbHIYLN41CLSaRCep5noBsofp3IAbka1nG9h+PCk7b\npoKmRh1PoTQY3gvmEm8JejL6JLOAmz3/aTQ2OW6FPnXWTH2vAc+/t7/pzRbm96+g\nrevRUJ/F7NKAWVa0d1gkk8HDaHwWnVD0oG972r3mbTCauuqTpTvba8D8QyTSuEFU\nejgLwuqx4pQXfPOHT0t7MCtfj48e7ZNaofc0al6QdczLJFKkilSEfFQ4TsoLfmH3\ns446tcIp5M9jsoO7atlL0YU0BGrgxkiWdEgACZu1AEsE3NqiPXVmDDWc55WIW5FG\nawIDAQAB\n-----END RSA PUBLIC KEY-----" }
}
}
```

## Registration document

This is an OPDS 2 document containing a single item of metadata, `shared_secret`.

```
{ "metadata": {
    "shared_secret": "jP3JSqRvU7IcJH/JarTBYH4Eg3Q4TSD1zRhr6VDrKzEAl2kGg2XC6ts3a7pUAH8+sjBrDmM2iyMc\nc7KQ2nr6obUp8geUUgsQ+MWnYk/Ti1Jpmx6Lo38ldTboqpykW+cFLH3iNg9JkaHgXI11dMsG+x4c\nTCcQ1u4yC3MT4ac3IgoJLcSUSmaNAX+fcze1s8DualXtBWUxGwWJprvYVVMVLnc6Z984QeSLDHSV\nlPlxW+DaBdUGmxvuO4RZp5Ld58cRMzyQn0xITXDJOmRGu+ZgRi1jB9Igje9x8cgwC9UurNnYJb7W\nk907jF3Yj0o8CC8/jkW5dIFla235M4/jZjnvSA=="
    }
}
```


# Collection management

Once you have registered a collection, you can use the `http://opds-spec.org/sort/new` link to get the delta feed. But there won't be anything in the delta feed until the metadata wrangler knows which titles are in your collection.

When you send an authenticated request to the Metadata Lookup Service, all the IDs you mention are implicitly added to your collection.

You can also use the http://librarysimplified.org/rel/metadata/collection-add and http://librarysimplified.org/rel/metadata/collection-remove links, to explicitly add and remove IDs from your collection.

# Metadata Lookup Service

This is a public service (at least for now) which lets you look up information about any supported ID (probably an ISBN) and get back an OPDS feed full of information.

# Name Resolution Service

This is a public service (at least for now) which takes an author name as you might see it on the front of a book, plus an optional identifier (identifying the book in question). It tries to return the author name as it would appear in a card catalog.