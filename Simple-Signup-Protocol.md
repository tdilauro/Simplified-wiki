The Simple Signup Protocol is a simple way to integrate a library's preexisting web-based signup form into that library's circulation manager, without requiring significant changes to the signup form. It is based on [[OAuth Implicit Grant|https://tools.ietf.org/html/rfc6749#section-4.2]], but the intent is to end up with a username and password that can be used with HTTP Basic Auth, not with an OAuth access token.

# Authentication for OPDS document

It all starts in the library's [[Authentication for OPDS|https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw/]] document, which the Library Simplified circulation manager serves along with a 401 (Unauthorized) response code. This document contains instructions explaining how to log in with your library card. But what if you don't _have_ a library card? That's where the `register` link comes in.

Here's an example Authentication for OPDS document with a `register` link.

```
{
  "id": "8e21cd8b-5075-4952-83c3-d37ac01df307",
  "title": "Public Library",
  "type": ["http://opds-spec.org/auth/basic"],
  "description": "Enter a valid library card number and PIN code to authenticate on our service.",
  "links": [
    "register": {"href": "http://example.com/registration", "type": "text/html"}
  ]
}
```

# Opening the web view

When an application (mobile or otherwise) sees a `register` link with a `type` of "text/html", it is allowed to open a web view to the URL in the link.

The client MUST modify the registration URL to include a `response_type` query parameter. The value of this parameter MUST be the literal string `client-password`. This tells the signup server that it's supposed to send back a login and password, rather than an OAuth Bearer Token or some other form of credentials.

The client MUST modify the registration URL to include a value for the `state` query parameter. The value of this parameter MUST be a randomly generated string. It serves the same security purpose as the `state` query parameter in OAuth's [["Authorization Request"|https://tools.ietf.org/html/rfc6749#section-4.2.1]] definition.

The client MUST modify the registration URL to include a `redirect_uri` query parameter. The value of this parameter MUST be an expansion of the following URI Template:

```
opds://authorize/{id}{?response_type,state}
```

The `id` parameter is REQUIRED. Its value MUST be the value of `"id"` found in the Authentication for OPDS document. It serves to inform the signup server which Authentication for OPDS document triggered the request for a new account.

Here's an example expansion of the URI Template based on the Authentication for OPDS document given above:

```
opds://authorize/8e21cd8b-5075-4952-83c3-d37ac01df307
```

Here's how that URI would be sent to the signup server linked to in the Authentication for OPDS document given above:

```
http://example.com/registration?response_type=client-password&state=594061549043850995&redirect_uri=opds%3A//authorize/8e21cd8b-5075-4952-83c3-d37ac01df307'
```

# The sign-up process

At this point the prospective patron goes through whatever web-based signup process the library uses. This process is invisible to the OPDS client.

# Back to the client

At some point, either the web view gets closed (because the patron has given up) or the web view is redirected to the `redirect_uri`. Here's the sort of HTTP response you might see from the signup server:

```
201 Created
Location: opds://authorize/8e21cd8b-5075-4952-83c3-d37ac01df307?login=1004005&password=9102&state=594061549043850995
```

When the client sees that the web view has been redirected to the `redirect_uri`, it MUST close the web view. 

# Understanding the result

The signup server communicates with the client by adding query parameters to the basic `opds://authorize/{id}` URI. This specification defines the meaning of the following parameters:

* `state` - This MUST be the value of `state` that was provided in the initial request to the signup server. This gives client confidence that the signup was processed by the right server.
* `login` - If the patron was issued a identifier (e.g. a username or barcode), or if an existing identifier for the patron was located, that identifier MUST be provided here. If no `login` is provided, it means that the process concluded without an identifier being associated with the patron -- perhaps the patron gave up on the process or it turned out they were not eligible.
* `password` - If the patron chose or was issued a password to go along with their identifier, it MAY be provided here. A library may choose not to send the password to the OPDS client, or may not know the password to send it. If that happens, the patron will have to enter their password manually.

If possible, the OPDS client MUST then use the information contained in the redirect URI to authenticate the patron (and complete the action they were trying to do when they got the original 401 error).

If the `state` doesn't match, it's an error condition that should be reported to the user.

# Conclusion

If the protocol is followed to completion, the patron now has a library card. The information they need to authenticate has been transferred to the OPDS client, and the OPDS client has logged them in.

If the protocol is not followed to completion, the OPDS client will know. Either the web view will be closed prematurely, or the final redirect URI will be missing a `login` parameter. These cases do not necessarily indicate an error condition and SHOULD NOT be treated as errors. Any error message that needed to be displayed, should have been displayed in the web view. 

If the protocol is not followed to completion, the patron should be given the same choices they were given before they started the process: to authenticate with the library, to (re)start the signup process with a fresh web view, or to cancel out and go back to the catalog.

# Noncompliant servers

It may not be possible to modify the signup server to support the final redirect. In fact, it may not be possible to modify the signup server at all. This protocol can still be useful, so long as the OPDS client allows the user to manually kill the web view. 

Remember, a prematurely killed web view is not an error condition. A user can launch the web view, sign up for a library card, manually kill the web view once they're done, and then log in with the barcode they were issued.

As such, an Authentication for OPDS document MAY contain a link with the relationship "register" even if the resource at the other end of the link does not support the Simple Signup Protocol.