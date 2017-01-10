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

The application SHOULD modify the registration URL to include a `redirect_uri` query parameter. This provides the equivalent of OAuth's ["redirection endpoint"|https://tools.ietf.org/html/rfc6749#section-3.1.2].

The application SHOULD modify the registration URL to include a `state` query parameter. This random string serves the same security purpose as the `state` query parameter in OAuth's [["Authorization Request"|https://tools.ietf.org/html/rfc6749#section-4.2.1]]] definition.

Example: `http://example.com/registration?state=594061549043850995&redirect_uri=opds://register`

# The sign-up process

At this point the prospective patron goes through whatever web-based signup process the library uses. This process is invisible to the OPDS client.

# Back to the client

At some point, either the web view gets closed (because the patron has given up) or the web view is redirected to the `redirect_uri`. Here's the sort of HTTP response you might see from the signup server:

```
201 Created
Location: opds://register?login=1004005&password=9102&state=594061549043850995
```

When the client sees that the web view has been redirected to the `redirect_uri`, it MUST close the web view. If possible, it MUST use the information contained in the `redirect_uri` to partially or completely authenticate the patron.

The signup server communicates with the client by adding query parameters to the `redirect_uri` it was given. This specification defines the meaning of the following parameters:

* `state` - If this was provided in the initial request to the signup server, it MUST be replicated in the `redirect_uri`. TODO: what if the client tries to use it and the server just doesn't support it?
* `login` - If the patron was issued a identifier (e.g. a username or barcode), or if an existing identifier for the patron was located, that identifier MUST be provided here. If no `login` is provided, it means that the process concluded without an identifier being associated with the patron -- perhaps the patron gave up on the process or it turned out they were not eligible.
* `password` - If the patron chose or was issued a password to go along with their identifier, it MAY be provided here. A library may choose not to send the password to the OPDS client, or may not know the password to send it. If that happens, the patron will have to enter their password manually.

# Conclusion

If the protocol is followed to completion, the patron now has a library card. The information they need to authenticate has been transferred to the OPDS client, and the OPDS client has logged them in.

If the protocol is not followed to completion, the OPDS client will know. Either the web view will be closed prematurely, or the final redirect URI will be missing a `login` parameter. This might not be an error condition and SHOULD NOT be treated as one. Any error message that needed to be displayed, should have been displayed in the web view. The patron should be given the same choices they were given before the process started: to authenticate with the library, to (re)start the signup process with a fresh web view, or to cancel out and go back to the catalog.

# Noncompliant servers

It may not be possible to modify the signup server to support `callback_url`. In fact, it may not be possible to modify the signup server at all. This protocol can still be useful, so long as the OPDS client allows the user to manually kill the web view. 

Remember, a prematurely killed web view is not an error condition. A user can launch the web view, sign up for a library card, manually kill the web view once they're done, and then log in with the barcode they were issued.

# Open questions

* Can `state` be used reliably?
* `client_id` is not necessary because the assumption is that a patron could just as easily sign up with their web browser or at a branch library.
* Is `code` necessary?
* Is `state` necessary?