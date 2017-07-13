Temporary Token Access Control (TTAC) is a simple "DRM" scheme which makes no effort to encrypt or obfuscate the resources it guards. Instead, access to the guarded resources is controlled by temporary tokens. If you have a token, you can send it as a bearer token to download the resource. This allows a library to delegate to its patrons, on a temporary basis, the ability to download a guarded resource such as an EPUB.

Other DRM systems use the [Client Token Protocol](https://github.com/NYPL-Simplified/Simplified/wiki/DRMAutodiscoverySpecs#the-client-token-protocol) to guide users through the registration process. With  TTAC, the Client Token Protocol is pretty much the whole thing. Once you have a client token, you can use it as a bearer token to request the guarded resource directly.

# Worked example

A publisher makes the following OPDS feed available to a library.

```
<feed>
<link rel="https://librarysimplified.org/rel/client-token"
           href="https://publisher-server/give-me-a-token"/>
<entry>
 <link rel="http://opds-spec.org/acquisition/"
       href="https://publisher-server/a-book.epub"
       type="application/epub+zip">
</entry>
</feed>
```

To see this feed or to download the EPUB it links to, the library must
provide its credentials (username "library", password "ilovebooks")
using HTTP Basic Auth.

That's great, as far as it goes, but the library isn't the entity that
will be downloading that EPUB. The library's patrons want to download
the EPUB, and the library shouldn't need to give out its credentials
("library:ilovebooks") to all of its patrons.

Temporary Token Access Control allows a library to obtain a temporary token
which it can pass on to a patron. The patron can use the token to
authenticate with the publisher and download books. Unlike
"library:ilovebooks", the token expires after a short period, making
it safe to give out to patrons.

Now, let's imagine a patron wants to download this EPUB. Instead of
going directly to the publisher, the patron authenticates with their
library. The library sends a request to the link with
`rel="https://librarysimplified.org/rel/client-token"`.

```
GET /give-me-a-token
Host: publisher-server
Authorization: Basic library:ilovebooks
```

The response looks like this:

```
HTTP/1.1 200 OK
Content-Type: vnd.librarysimplified/drm-client-registration-token?scheme=http://librarysimplified.org/terms/drm/scheme/TTAC
Expires: Thu, 13 Jul 2017 18:01:44 GMT

Wt='{&Vm0a
```

This says that `Wt='{&Vm0a` is a valid client token. The server passes
this information to the library patron (TODO: how?), and the library
patron is (temporarily) able to make an authenticated request for the
EPUB.

```
GET /a-book.epub
Host: publisher-server
Authorization: Bearer Wt='{&Vm0a
```

# Link relation

When TTAC is in use, the Client Token Protocol endpoint is indicated
by a link with the relation
`https://librarysimplified.org/rel/content-token`.

An OPDS server that supports TTAC SHOULD link to the corresponding CTP
endpoint from any OPDS feed containing an <entry> that can be
fulfilled through this scheme.

# Client Token Protocol behavior

Using the [Client Token Protocol](DRMAutodiscoverySpecs#the-client-token-protocol) to get a client token for TTAC works the same way as getting a client token for use in some other DRM system. You make an authenticated request to the protocol endpoint and get a token in response. In the worked example above, this is the request to `https://publisher-server/give-me-a-token`.

To indicate that a client token should be used as part of TTAC, the
server MUST serve it as the media type
`vnd.librarysimplified/drm-client-registration-token?scheme=http://librarysimplified.org/terms/drm/scheme/TTAC`.

#  Lifetime

Since client tokens generated as part of TTAC are directly used to get
access to guarded resources, there are some extra rules surrounding
them that don't apply to Client Token Protocol tokens in general.

A client token MUST be reusable until it expires. A clients MUST NOT use
a token it knows to be expired.

A client token MUST be valid for least ten seconds after issuance.

When the Client Token Protocol endpoint serves a client token for use
in TTAC, it SHOULD set the `Expires` to the time at which the token
expires. If the `Expires` header is not present, a TTAC client SHOULD
assume that a token expires ten seconds after the request for the
token was sent.

There is no requirement that the Client Token Protocol endpoint return
a unique token with every request. A server may reuse tokens
indefinitely if it's comfortable with that level of security.

# Do not cache

The server that serves a client token MUST set the `Cache-Control`
header to `private`. This tells HTTP intermediaries not to cache the
token. (This should be in the Client Token Protocol spec, actually.)

