In an OPDS 2 Catalog, you may encounter a link with a link relation of
"register" and a media type of
`application/opds+json;profile=https://librarysimplified.org/rel/profile/directory`:

```
"links": [
 {"rel": "register",
  "type": "application/opds+json;profile=https://librarysimplified.org/rel/profile/directory",
  href="https://example.com/opds-directory"}
]
```

This link is an entry point to the OPDS Directory Registry Protocol
(below, just "Protocol"). It says that the OPDS Catalog can be seen
not just as a list of books (if there are any books at all) but as a
directory of _other OPDS servers_. If you yourself run an OPDS server,
you may be able to register it with the directory by going through this
Protocol.

By contrast, a link with the `register` link relation but a media type
of `text/html` is probably a link for letting a human being sign up
for an account on the OPDS server.

# Kicking off the registration process

Send a POST request to the `register` link with an
`application/x-www-form-urlencoded` form as payload. The form MUST
contain a single key-value pair, `url`, with the value being the URL
to your OPDS server.

```
POST /opds-directory
Host: example.com
Content-Type: application/x-www-form-urlencoded

url=http://example.org/my-opds-server/
```

The directory will retrieve the root feed of your OPDS server and try
to get the information it needs to add you to the directory.

If this information is missing, or the directory can't make sense of
it, you'll get a [problem detail
document](https://tools.ietf.org/html/rfc7807). If the registry is
able to acommodate your OPDS server, you'll get a registration
document with the media type
`application/opds+json;profile=https://librarysimplified.org/rel/profile/directory`. This
document (discussed below) shows you what the directory was able to
derive from the information you gave it.

# What the directory expects

When you trigger the registration process, the directory will send an
unauthenticated GET request to the URL you specified. It's expecting
to get one of the two results:

* A 200 response code with an OPDS 1 or OPDS 2 catalog in the entity-body.
* A 401 response code with an [Authentication for OPDS document]()
  (media type application/vnd.opds.authentication.v1.0+json) in the
  entity-body.

If the directory gets an OPDS catalog, it will look inside the catalog
document for links with the relation `authenticate`. It will send GET
requests to `authenticate` links until it finds one that returns an
Authentication for OPDS document.

One way or the other, the directory expects to be able to fetch an
Authentication For OPDS document from your OPDS server.

If your server sends a 401 response code and no Authentication for
OPDS document, registration will fail. You've given no indication that
proper authentication will actually grant access to an OPDS catalog.

If you serve an OPDS catalog that doesn't link to an Authentication
For OPDS document, registration will fail. There's not enough
information in an ordinary OPDS catalog to create a directory entry
that will help people find the catalog they're looking for.

Even an OPDS server that allows anonymous access can have an
Authentication For OPDS document. In fact, that document is how you
explicitly tell the directory that your OPDS server allows anonymous
access.

# What the directory looks for

At this point the directory has an Authentication for OPDS document
from your server. It may also have an OPDS catalog. Any of the
information in these documents may be used to build your entry in the
directory.

These two fields MUST be present in your Authentication for OPDS
document, or registration will fail:

* `id`: The URL to the OPDS root catalog.
* `title`: The name of the collection.

The value for `id` must match the URL you originally sent to the
Protocol endpoint. If it doesn't match, your registration will fail.

Although it's not required, your Authentication For OPDS document
SHOULD include a `logo` link that points to the logo associated with
the institution providing the OPDS server.

Specific directory implementations may gather additional information
and impose additional requirements.

## SimplyE extensions

The SimplyE directory also looks in your Authentication for OPDS
document for a number of extensions from [this
list](https://github.com/NYPL-Simplified/Simplified/wiki/Authentication-For-OPDS-Extensions).

* [`service_description`](Authentication-For-OPDS-Extensions#server-description)
* [`color_scheme`](Authentication-For-OPDS-Extensions#color-scheme)
* [`collection_size`](Authentication-For-OPDS-Extensions#collection-size)
* [`audiences`](Authentication-For-OPDS-Extensions#audiences)
* [`service_area`](`Authentication-For-OPDS-Extensions#service_area`)
* [`focus_area`](`Authentication-For-OPDS-Extensions#focus_area`)
* [`public_key`](Authentication-For-OPDS-Extensions#public-key) is
  is used to negotiate a shared secret, a process covered separately
  below.

The SimplyE directory also looks for some other things in your
Authentication For OPDS document which aren't strictly extensions.

* We expect to see [a link to the homepage of the institution that
  runs the
  server](Authentication-For-OPDS-Extensions#rel-alternate-type-texthtml)
* If applicable, we expect to see [a link that a human being can use
  to apply for access to the OPDS
  server.](Authentication-For-OPDS-Extensions#rel-register) Note that
  although the link relation here is `register`, this link is _not_ supposed
  to be an entry point into the OPDS Directory Registration Protocol.
  That's a machine-to-machine integration for putting an OPDS server
  into a directory; this is used by a human being to get an
  account on an OPDS server.
* If your OPDS server allows anonymous access, we expect to see that
  [included as one of the authentication
  methods](Authentication-For-OPDS-Extensions#link-relation-for-anonymous-access).
* The SimplyE directory has [special additional
  guidelines](Authentication-For-OPDS-Extensions#rellogo) on the size
  and appearance of the logo.

All this extra stuff is optional, and omitting it will not cause your
SimplyE registration to fail. However, omitting this information may
give potential users an inaccurate picture of your OPDS server.

For example, omitting `service_area` will give SimplyE users the
impression that everyone in the world is free to download content from
your OPDS server. Sometimes this is accurate! But if it is accurate,
it's better to explicitly state this by putting `"service_area":
"everywhere"` in your Authentication For OPDS document.

The goal here is to guide people to OPDS servers that they can
actually use to get books. We want to guide people living in Detroit
to the Detroit Public Library and not to the Kenosha Public Library.

# Negotiating a shared secret

You may find it useful to establish a shared secret with the OPDS
directory. A shared secret allows you to prove your ownership of an
OPDS server when communicating with the directory after registration.

A shared secret will also allow you to create JSON Web Tokens or
[Short Client Tokens](Short-Client-Token) that the directory can
verify. This gives the directory the ability to validate your users'
credentials without knowing anything about them or being able to use
your authentication system as an oracle.

To indicate that you want to establish a shared secret, generate an
RSA key and put the public key in your Authentication For OPDS
document as `public_key`. Here's an example:

```
"public_key": { "type": "RSA", "value": "-----BEGIN RSA PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA6dM2j3D3ykxLfbZ4zm1o\nx6wzSvbUsL6rrdTQ+J/zduBbHIYLN41CLSaRCep5noBsofp3IAbka1nG9h+PCk7b\npoKmRh1PoTQY3gvmEm8JejL6JLOAmz3/aTQ2OW6FPnXWTH2vAc+/t7/pzRbm96+g\nrevRUJ/F7NKAWVa0d1gkk8HDaHwWnVD0oG972r3mbTCauuqTpTvba8D8QyTSuEFU\nejgLwuqx4pQXfPOHT0t7MCtfj48e7ZNaofc0al6QdczLJFKkilSEfFQ4TsoLfmH3\ns446tcIp5M9jsoO7atlL0YU0BGrgxkiWdEgACZu1AEsE3NqiPXVmDDWc55WIW5FG\nawIDAQAB\n-----END RSA PUBLIC KEY-----" }
```

Currently the only supported key type is `RSA`.

If the directory supports shared secret negotiation, it will sign a
randomly generated string with your public key, encode it using base64
encoding, and send it to you as `shared_secret` in the registration
document. Decode it, decrypt it using your public key, and you have
the shared secret.

A directory is not required to support shared secret negotiation. If a
directory doesn't support shared secret negotiation, your `public_key`
will be ignored, and the registration document you get after
registration will not include a `shared_secret`.

# The registration document

The document returned on a successful registration is an OPDS 2
Catalog that describes your entry in the OPDS directory. It is served
as the media type
`application/opds+json;profile=https://librarysimplified.org/rel/profile/directory`;
that is, as an OPDS 2 catalog with a profile specific to this Protocol.

In general, this catalog contains the information potential users will see when they encounter your library in the directory. However, there are two extra fields that will only be shown to you. Their semantics are defined in the profile definition below.

```
{
 "metadata" : {
   "short_name": "YOURLIBR",
   "shared_secret": "jP3JSqRvU7IcJH/JarTBYH4Eg3Q4TSD1zRhr6VDrKzEAl2kGg2XC6ts3a7pUAH8+sjBrDmM2iyMc\nc7KQ2nr6obUp8geUUgsQ+MWnYk/Ti1Jpmx6Lo38ldTboqpykW+cFLH3iNg9JkaHgXI11dMsG+x4c\nTCcQ1u4yC3MT4ac3IgoJLcSUSmaNAX+fcze1s8DualXtBWUxGwWJprvYVVMVLnc6Z984QeSLDHSV\nlPlxW+DaBdUGmxvuO4RZp5Ld58cRMzyQn0xITXDJOmRGu+ZgRi1jB9Igje9x8cgwC9UurNnYJb7W\nk907jF3Yj0o8CC8/jkW5dIFla235M4/jZjnvSA=="
  }
}
```

# Manual approval step

An OPDS directory may automatically publish every server that
registers with it, but it is more likely to require a manual approval
step. The SimplyE OPDS directory has a manual approval step to verify
that the OPDS server is compatible with SimplyE, and that the server
is provided by a bona fide public library or similar institution.

# Updating your registration

To update your registration, repeat the POST request you sent to the
`register` link that led to you registering in the first place. The
directory will retrieve updated information from your OPDS server and
try to update your entry in the directory.

If something goes wrong, you'll get a problem detail document and your
registration will not be updated. If everything goes well, you'll get
a new registration document that reflects your current registration.

If you want to change your shared secret, include the current `shared_secret` as a bearer token
when you submit the form.

```
POST /opds-directory
Host: example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer bf58b099a8dd250e617b11a583ba9186c5520eaeef9e01c2

url=http://example.org/my-opds-server/
```

The registration document you get back will have your new
`shared_secret`.

There's no need to provide your shared secret unless you want it changed. For normal re-registration, the fact that you control the OPDS server is sufficient to cause an update. Someone else can forge a re-registration request for your OPDS server, but since you control the documents served from that server, nobody can make your directory entry contradict the information published by your server.

# Removing your registration

TBD

# Profile definition

The `https://librarysimplified.org/rel/profile/directory` profile
defines three additional fields that may show up in the `metadata`
section of an OPDS 2 catalog.

* `adobe_vendor_id`: The Adobe Vendor ID assigned by Adobe to this
  directory. If you are sending a Short Client Token to Adobe in hopes
  of obtaining an Adobe Client ID from this directory, use the
  `adobe_vendor_id` value as the vendor ID.

* `short_name`: The short name assigned to an OPDS catalog by the
  directory. You should use this as the "Library name" when generating
  a Short Client Token to be sent to the directory.

* `shared_secret`: A shared secret between your OPDS catalog and the
  directory. This string is base64-encoded and encrypted using your
  `public_key`. You'll need to decode and decrypt it before using it.

The `shared_secret` and `short_name` values only show up in the
document returned after a successful registration. The
`adobe_vendor_id` value should show up in the root OPDS catalog on the
directory server, alongside the `register` link that points to the
Protocol endpoint.