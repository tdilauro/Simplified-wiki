# The DRM XML namespace

This namespace is used to convey information about the DRM scheme used
to encrypt documents served through OPDS feeds, and to provide the
information necessary for a client to download and decrypt those
documents.

The URI for this namespace is
`http://librarysimplified.org/terms/drm`. It is sometimes prefixed as
`drm` in examples, e.g. `drm:clientToken`.

## `drm:drm`

The `drm:drm` tag indicates that some special DRM-specific information
will help the client decode a representation. In some cases, it may
not be possible to even retrieve a representation without the
information kept in the `drm:drm` tag.

This document defines the semantics of the `drm:drm` tag when the value of
the `scheme` attribute is one defined in the DRM Scheme Registry. It
does not define the semantics of the `drm` tag when the value of the
`scheme` attribute is a URI not found in the DRM Scheme Registry.

The `drm:drm` tag MAY be the child of an `atom:link` tag. When it appears,
it is intended to shed light on the representation that will be
returned by following that link.

The `drm:drm` tag MUST provide a value for the `drm:scheme`
attribute. Depending on the DRM scheme in use, values for the `drm:clientToken` and
`drm:serverToken` attributes might be required, optional, or forbidden.

## `drm:scheme`

The value of the `scheme` attribute identifies the DRM scheme in
use. This MUST either be a short string from the DRM Scheme Registry
(q.v.), or a URI.

When the `drm:drm` tag is used, it MUST provide a value for the
`drm:scheme` attribute.

When `atom:link` is used to link to a resource with the `atom:rel` of
"http://librarysimplified.org/terms/drm/register-client", the
`atom:link` tag MUST also provide a value for `drm:scheme`.

## `drm:clientToken`

The value of `drm:clientToken` is a piece of information that may be
necessary or useful in authenticating the client with the DRM server,
authorizing the client to obtain the DRM-encrypted resource, or
decrypting the resource once obtained.

When the Adobe ACS DRM scheme is in use, the value of
`drm:clientToken` MUST be interpreted as `authData` that can be used
to obtain an Adobe ID. If the client already has an Adobe ID, the
value of `drm:clientToken` MAY be ignored. If no value for
`drm:clientToken` is provided, the client MAY proceed as usual, but
may be unable to download and decrypt the ACS-encrypted resource.

When the Readium LCP DRM scheme is in use, the value of
`drm:clientToken` MUST be interpreted as the LCP user key that guards
the resource. If the client knows the User Passphrase, the value of
`drm:clientToken` MAY be ignored. If no value for `drm:clientToken` is
provided, the client MUST prompt for the User Passphrase as per the
LCP spec..

When the URMS DRM scheme is in use, the server SHOULD NOT provide a
value for the `drm:clientToken` attribute. If this value is present it
MUST be interpreted as the URMS CCID, but in a compliant system this
will not provide any additional information, because a URMS CCID is
given to OPDS clients through the `urms-ccid` URI scheme (q.v.).

## `drm:serverToken`

The value of `drm:serverToken` is a piece of information that may be
useful in distinguishing between multiple providers of the same
information. The meaning of the attribute differs depending on the DRM
scheme in use.

When the URMS DRM scheme is in use, the value of `drm:serverToken`
serves to identify the URMS Store that provides the resource.

When the LCP or Adobe DRM schemes are in use, the value of
`drm:serverToken` is undefined, and a server SHOULD NOT provide it.

# The DRM Scheme Registry

This specification serves as a registry for reserved strings
corresponding to popular DRM schemes. The following schemes are
defined:

| Short string | URI | Description |
| ACS | http://librarysimplified.org/terms/drm/scheme/ACS | Adobe ACS |
| URMS | http://librarysimplified.org/terms/drm/scheme/URMS | Sony URMS |
| LCP | http://librarysimplified.org/terms/drm/scheme/LCP | Readium URMS |

# The `http://librarysimplified.org/terms/drm/register-client` link relation

This link relation is used to indicate a URL that can kick off the
process of registering a client with a DRM server. An `atom:link` that
uses this link relation MUST also include the `drm:type`
attribute. (TODO: Or we could define two different link relations.)

When the ACS DRM scheme is in use, a client which sends a properly
authenticated HTTP GET request to the target URL MUST get back a
representation of media type `text/plain`. The entity-body MUST
contain a Base64-encoded string that can be used as authData, as per
the Adobe Vendor ID Specification, to get an Adobe ID.

When the URMS DRM scheme is in use, a client which sends a properly
authenticated HTTP GET request to the target URL MUST get back
representation of media type `text/plain`. The entity-body MUST
contain the `authToken` portion of a request to the URMS "Generate
Authtoken" API endpoint (as per the URMS Store API specification) for
the authenticated user. For example, if the entity-body of the
response to the "Generate Authtoken" endpoint looks like this:

```
{
       "statusCode": 0,
       "message": "Success.",
       "authToken": "3375:z4nk82tdj32hf4ad"
}
```

Then the entity-body of the response to the `drm/register-client`
request would look like this:

```
3375:z4nk82tdj32hf4ad
```

If the authenticated user is not registered with the URMS store, the
server MUST register them (using the "Register User" endpoint) before
attempting to generate an `authToken`.

This specification does not define the expected behavior of a resource
pointed to by this link relation when any other DRM scheme is in use,
or when `drm:scheme` is missing from the link.

# The `vnd.librarysimplified/obfuscated` media type

This media type is used to describe a document that has been
obfuscated or encrypted according to a DRM scheme or some other
algorithm. The document may be served as some other media type, such
as `application/epub`, and may even be recognizable as a file of that
type, but cannot be properly rendered until it is decrypted or
deobfuscated.

This media type defines two optional parameters:

*scheme*: A non-empty list of space-separated URIs identifying the
algorithm(s) used to encrypt or obfuscate the representation. URIs
found in the DRM Scheme Registry are especially appropriate values for
*scheme*. If multiple URIs are present, they should be listed in the
order they were applied to the original representation.

*original*: The name of a media type, representing the
representation's original media type before encryption and obfuscation
were applied.

# The `urms-ccid` URI scheme

# atom:link inside atom:link