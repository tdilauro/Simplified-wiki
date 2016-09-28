# DRM Extensions to OPDS

This spec defines a new XML namespace. Elements from this namespace can be included in an OPDS `<link>` tag to convey information about the DRM scheme used to encrypt the document at the other end of the link. The purpose is to provide the information necessary for a client to download and decrypt those documents, and to inform clients of the DRM scheme in use, so that clients unable to handle that DRM scheme can bow out gracefully.

The URI for this namespace is
`http://librarysimplified.org/terms/drm`. In the examples that follow it is prefixed as
`drm` in examples, e.g. `drm:clientToken`.

This namespace defines three tags (`drm`, `clientToken`, and `serverToken`), and two attributes (`scheme` and `href`).

## `drm:licensor`

The `drm:licensor` tag indicates that some special DRM-specific licensing information will help the client decode a representation. In some cases, it may not be possible to even retrieve a representation without the information kept in the `drm:licensor` tag.

This document defines the semantics of the `drm:licensor` tag under DRM systems described in the DRM Scheme Registry (q.v.). It does not define the semantics of the `drm:licensor` tag under other DRM systems.

The `drm:licensor` tag MAY be the child of an `atom:link` tag. When it appears, it is intended to shed light on the representation that will be returned by following that link.

If acquiring a book through an `<opds:link>` tag would require negotiating a DRM system, the client MUST be able to determine _which_ DRM system is in use by examining the `<opds:link>` tag and its children. It MUST be possible to determine which DRM system is in use _before_ triggering an unsafe state transition such as OPDS `borrow` or `buy`. There are three ways to determine which DRM system is in use:

* A DRM-specific media type, either in the `type` of the `<opds:link>` tag, or in an embedded `<opds:indirectAcquisition>` tag, can indicate that the resource is controlled by an ACS or LCP system. For example, a link that leads to an `vnd.adobe/adept+xml` file which leads to an `application/epub+zip` file is controlled by an ACS system.

* A `ccid-urms` URL in the target of an `<opds:link>` indicates that the resource is controlled by a URMS system.

* The use of the `vnd.librarysimplified/obfuscated` media type in the `<opds:link>` tag or an embedded `<opds:indirectAcquisition>` indicates that a DRM system is in use. The `scheme` parameter to that media type indicates _which_ DRM scheme is in use.

It's important for two reasons to identify the DRM scheme in use. First, most clients can handle some DRM schemes but not others. A client has a right to know, before asking its user to pay for a book, whether it will actually be able to decrypt the book. Second, some of the `drm:` elements and attributes have different meanings, depending on the DRM scheme in use.

## `drm:clientToken`

The `drm:clientToken` tag either contains or links to a piece of information that may be
necessary or useful in authenticating or authorizing the client with the DRM server. This information might be necessary to obtain the DRM-encrypted resource, or to decrypt it after obtaining it.

The `drm:clientToken` tag MAY contain a string leaf node. If present, the string value of the leaf node MUST be interpreted as a client token according to the Client Token Protocol.

The `drm:clientToken` tag MAY contain a `drm:href` attribute. If present, the value of `drm:href` MUST be an URL with the `https:` scheme. This URL MUST comply with the Client Token Protocol, and its `vnd.librarysimplified/drm-client-registration-token` representation MUST be interpreted as a client token according to the Client Token Protocol.

The meaning of the client token obtained from a `drm:clientToken` tag depends on the DRM scheme in use for the enclosing `<opds:link>`.

## `drm:serverToken`

The `drm:serverToken` tag contains information that may be useful in distinguishing between multiple DRM servers of the same type.

The `drm:serverToken` element MAY contain a string leaf node.

The `drm:serverToken` MAY contain a value for the `drm:href` attribute.

The meaning of the values associated with the `drm:serverToken` tag depends on the DRM scheme in use for the enclosing `<opds:link>`.

### `serverToken` under URMS

When the URMS DRM scheme is in use, a `drm:serverToken` tag SHOULD be provided. If it is provided, both `drm:href` and a leaf node MUST be present.

The value of `drm:href` is the URL of the URMS Store that provides the resource. The string leaf node is the ID of the URMS Store that provides the resource.

### `serverToken` under other DRM schemes

When the LCP or Adobe DRM schemes are in use, the meaning of the `drm:serverToken` tag is undefined. It SHOULD NOT be provided.

# The Client Token Protocol

A URL that complies with the Client Token Protocol MUST respond to a properly authenticated HTTP GET request with  a client token.

This specification defines the meaning of client tokens of media type `vnd.librarysimplified/drm-client-registration-token`. It does not define the meaning of client tokens of other media types.

If the media type of a client token is `vnd.librarysimplified/drm-client-registration-token`, the `scheme` parameter MUST be set to the DRM scheme in use.

The meaning of a client token depends on the DRM scheme in use. This specification defines the meaning of a client token for the DRM schemes defined in the DRM Scheme Registry.

### Client token under ACS

When the Adobe ACS DRM scheme is in use, the client token MUST be interpreted as base64-encoded `authData` that can be used to obtain an Adobe ID. If the client already has an Adobe ID, the client token MAY be ignored.

If no client token is provided, and the client has no Adobe ID, the client MAY proceed with fulfilling a book as usual, but will most likely be unable to download or decrypt the ACS-encrypted resource.

### Client token under URMS

When the URMS DRM scheme is in use, the client token MUST be interpreted as an AuthToken like the one generated by the URMS "Generate Authtoken" endpoint. For example, if the entity-body of the
response to the "Generate Authtoken" endpoint looks like this:

```
{
       "statusCode": 0,
       "message": "Success.",
       "authToken": "3375:z4nk82tdj32hf4ad"
}
```

Then the client token would look like `3375:z4nk82tdj32hf4ad`.

Since generating a URMS Authtoken is an expensive operation, it is recommended that URMS client tokens be retrieved on demand (through the URL in the `drm:href` attribute) rather than provided in advance (through the `drm:token` attribute).

### Client token under LCP

When the Readium LCP DRM scheme is in use, the client token MUST be interpreted as the LCP user key that guards
the resource. If the client knows the User Passphrase, the client token MAY be ignored -- the client can hash the User Passphrase and get the same result. If no client token is provided, the client MUST prompt for the User Passphrase as per the LCP spec.

If the provider serves `drm:client-token` upon initial checkout, it MUST also serve the same `drm:client-token` every time it describes the loan (e.g. when listing books on the patron's bookshelf). This way, a patron will always be able to bring a book onto a fresh device and read it there.

In a library setting, the LCP user key SHOULD be a different value for every loan. This suggestion takes precedence over the statement in 4.4 of the LCP spec that "the Provider should use the same User Key for all licenses issued to the same User." This allows a library to evade a major privacy problem: the existence of a persistent identifier (such as the Adobe ID or URMS client ID) associated with every one of a patron's loans and tracked outside the library's control.

(The downside of providing a different user key for every loan is that the patron will be unable to bring their book into an e-reader application that does not also support OPDS and this DRM autodiscovery protocol.)

# The DRM Scheme Registry

This specification serves as a registry for reserved strings and URIs
corresponding to popular DRM schemes. The following schemes are
defined:

| Short string | URI | Description |
| ------------ | --- | ----------- |
| ACS | http://librarysimplified.org/terms/drm/scheme/ACS | Adobe ACS |
| URMS | http://librarysimplified.org/terms/drm/scheme/URMS | Sony URMS |
| LCP | http://librarysimplified.org/terms/drm/scheme/LCP | Readium LCP |

# The `vnd.librarysimplified/obfuscated` media type

This media type is used to describe a document that has been
obfuscated or encrypted according to a DRM scheme or some other
algorithm. The document may be served as some other media type, such
as `application/epub+zip`, and may even be recognizable as a file of that
type, but cannot be properly processed or understood until it is decrypted or
deobfuscated.

This media type defines two optional parameters:

*scheme*: A non-empty list of space-separated URIs identifying the
algorithm(s) used to encrypt or obfuscate the representation. URIs
found in the DRM Scheme Registry are especially appropriate values for
*scheme*. If multiple URIs are present, they should be listed in the
order they were applied to the original representation.

*original-type*: The name of a media type, representing the
representation's original media type before encryption and obfuscation
were applied. To avoid ambiguity, this media type may not have parameters applied to it. (TODO: URL-escaping the parameter value would allow parameters to be applied to it.)

# The `vnd.librarysimplified/drm-client-token` media type

A document of this media type represents a currently active client token (see "`drm:clientToken`"). Apart from this fact, its semantics are identical to the semantics of `application/octet-stream`.

This media type defines one required parameter:

*scheme*: A URI identifying the DRM scheme associated with the client token.
URIs found in the DRM Scheme Registry are especially appropriate values for
*scheme*. For example, an LCP user key would be served as vnd.librarysimplified/drm-client-token;scheme=http://librarysimplified.org/terms/drm/scheme/LCP.

# The `ccid-urms` URL scheme

The `ccid-urms` URL scheme is a way of linking to a URMS-encrypted resource by its CCID. The format is `ccid-urms:{ccid}`, where `{ccid}` is the CCID.

Example: `ccid-urms:000000000000000000000000000001` refers to the URMS resource with CCID "000000000000000000000000000001".

A `ccid-urms` URL is dereferenced by downloading (if necessary) and decrypting the URMS resource identified by the CCID. Generally this means extracting the CCID and passing it into the `RegisterBookTask` method of the URMS SDK, or the equivalent method of a compatible SDK.

If the resource is cached locally, a `ccid-urms` URL may be dereferenced by decrypting the cached resource, without downloading it again.

The `ccid-urms` scheme introduces no special security considerations beyond those introduced by the URMS system itself.

The `ccid-urms` scheme does not define semantics for the fragment identifier.

The name of the scheme was chosen to be `ccid-urms`, rather than `urms-ccid`, because of Section 2.5 of [RFC 2718](https://tools.ietf.org/html/rfc2718).

# atom:link inside atom:link

In rare cases it may be necessary to insert an `<atom:link>` tag inside another `<atom:link>` tag. Although this is valid Atom, the Atom RFC does not define the meaning of that second `<atom:link>` tag.

We interpret an embedded `<atom:link>` tag to be a link from _the resource in the parent link_ to some other resource. The `rel` of the embedded link is the relationship between _the resource in the parent link_ and the other resource, _not_ the relationship between the Atom entry and the other resource.

Here's an example:

```
<entry>
 ...
 <link rel="acquisition"
       href="ccid-urms:01234567890"
       type="vnd.librarysimplified/obfuscated;method=http://librarysimplified.org/terms/drm/URMS;original-type=application/epub"
  >
  <opds:indirectAcquisition type="application/epub"/>
  ...
  <link rel="acquisition"
        href="https://host/foo.epub"
        type="vnd.librarysimplified/drm-encrypted;method=urms;decrypts-to=application/epub"
  />
 </link>
 ...
</entry>
```

The `<entry>` is talking about a certain book. The parent link is to the resource `ccid-urms:01234567890`. The link relation is `acquisition`, indicating that dereferencing `ccid-urms:01234567890` will get you an (obfuscated) copy of the book. The `<opds:indirectAcquisition>` tag inside this first link indicates that the client can, if it knows how, deobfuscate the book and turn it into a normal `application/epub` document.

Inside that link is another link, to `https://host/foo.epub`. Again, the link relation is `acquisition`. This indicates that acquiring `https://host/foo.epub` is a _partial_ solution to acquiring `ccid-urms:01234567890`. It's not a complete solution, because you're going to end up with an obfuscated copy of the book -- there's no `<opds:indirectAcquisition>` tag inside the child link, the way there is inside a parent. The only way to turn the obfuscated book into a normal `application/epub` is to properly dereference the `ccid-urms` URL.

So why bother? Because giving the path to the obfuscated file allows the client to download it in the background, and fulfill it later. This is totally optional but it can speed things up.

Putting the HTTP link inside the `ccid-urms` link makes it clear that they represent different routes to the same fulfillment. Keeping them as separate links would make it look like were two different formats of the book: one that is permanently obfuscated, and one that is obfuscated but can be turned into an EPUB.