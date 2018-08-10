# Summary

The User Profile Management Protocol (hereafter the "Protocol") is a very simple HTTP-based protocol for retrieving information about a user profile and modifying that user's account settings. Although designed for use in managing patron accounts for public libraries, the protocol may be used to manage other types of user accounts.

## Who should implement?

Any library that wants to participate in the SimplyE program should implement the Protocol. The Library Simplified circulation manager implements the Protocol, so most SimplyE participating libraries don't have to do anything.

A library may choose not to implement the Protocol. SimplyE will revert to sensible defaults.

Other organizations may implement the Protocol for other types of user profiles, by defining their own keys for the fields.

# The `http://librarysimplified.org/terms/rel/user-profile` link relation

In the examples to follow, I'll use `http://server/settings` as the URL of a resource that implements the Protocol.

We introduce the link relation `http://librarysimplified.org/terms/rel/user-profile`, which indicates that the link target supports the Protocol. This link relation may be used anywhere.

Here's an HTML example of a link into the Protocol:

```
<a href="https://server/settings" rel="http://librarysimplified.org/terms/rel/user-profile" type="vnd.librarysimplified/user-profile+json">
 Change your settings
</a>
```

# Behavior under HTTP

This is mostly common-sense HTTP stuff.

## Authentication

Unauthenticated users may not participate in the Protocol. An unauthenticated request to an Protocol endpoint MUST result in a 401 response code with an authentication demand.

The settings in a Protocol document apply across the authentication domain to which the Protocol server belongs. These settings do not necessarily apply to other authentication domains, even other domains on the same server.

In general, a user may only administer their own settings. Users with superuser privileges MAY use the Protocol to administer settings for other users, but this spec does not define how that might work.

## GET

An Protocol server that receives an authenticated GET request SHOULD send a document of media type `vnd.librarysimplified/user-profile+json`. It MAY send some other media type that supports the same essential features.

## PUT

An authenticated PUT request to a Protocol endpoint SHOULD be accompanied by a document of media type `vnd.librarysimplified/user-profile+json`. It MAY be accompanied by a document of some other media type that supports the same essential features. The Protocol server MUST reject a document it does not understand.

Upon receipt, the server MUST do its best to make the underlying account settings reflect the content of the incoming document (see below). If this is not possible, due to a security violation, nonsensical values, conflicting values, or any other reason, the server MUST send a [[problem detail|https://tools.ietf.org/html/rfc7807]] explaining the problem.

If the client making a PUT request omits a setting from its payload, the server MUST interpret this as a wish that the value of that setting remain unchanged. If the client making a PUT request provides a value of `null` for a setting, the server MUST interpret this as a wish that the setting be set to a null value.

# The `vnd.librarysimplified/user-profile+json` media type

A document with the media type `vnd.librarysimplified/user-profile+json` (hereafter "Protocol document") represents the current state of a user's profile and account settings (when sent from the server to the client) or a desired future state of the user's account settings (when sent from the client to the server).

This document has the form of a single JSON object. With a few noted exceptions (`links` and `settings`), the keys and values inside this object correspond to pieces of information associated with the user's profile.

This specification defines semantics for the following keys:

* The special keys `links` and `settings`.
* The keys in the "Profile Registry" below.

When a Protocol document is sent along with a PUT request, the contents of the document SHOULD be ignored except for the value associated with the `settings` key.

## Examples

This example might be included with a GET response. It conveys one piece of information which the user cannot change (the amount of their fines) and one piece of information that the user can change (whether or not their reading activity is synchronized with the server).

```
{
 "simplified:fines": {"amount": "4.23", "currency": "USD"},
 "settings": { "simplified:synchronize_annotations" : false }
}
```

This example might be included with a PUT request. It signifies an
intent to change the value of `simplified:synchronize_annotations` to
true.

```
{
 "settings": { "simplified:synchronize_annotations" : true }
}
```

## `settings`

The value of `settings` MUST be a JSON object. The keys and values inside this object correspond to pieces of information associated with the user's profile. It works just like the root JSON object, except that the special keys `links` and `settings` have no defined meaning.

When a server sends a Protocol document in response to a GET request, the presence of a key in `settings` (as opposed to the root object) indicates that the client MAY attempt to change the value associated with that key by sending the new value as part of a PUT request.

When a client sends a Protocol document as part of a PUT request, the presence of a key in `settings` indicates that the client _is in fact_ attempting to change the value associated with that key to the value given in the Protocol document.

## `links`

The `links` key is reserved as a place to put hypermedia
links. The format of the value associated with `links` is currently undefined.

## `drm`

The `drm` key is reserved as a place to put DRM keys. If present, the value associated with `drm` MUST be a list of objects.

## Other notes

This specification does not define a general mechanism for conveying the human-readable names, descriptions, or possible values of profile elements.

In general, this specification does not define whether or not a given setting is writable (and thus, whether it belongs in the root object or in the `settings` sub-object). That depends on the application.

# Profile registry

Semantics for the following profile elements are defined. 

The `schema` namespace is reserved. All settings whose names start with "schema:" have semantics defined by the appropriate entry on schema.org. For instance, the value of "schema:givenName" is defined, as per [[https://schema.org/givenName|https://schema.org/givenName]], as the given name of the authenticated user.

The `simplified` namespace is reserved. All elements whose names start with "simplified:" will have their semantics defined in this section.

The `drm` namespace is reserved. All elements whose names start with "drm:" will have semantics defined by the [DRM Extensions to OPDS](https://github.com/NYPL-Simplified/Simplified/wiki/DRMAutodiscoverySpecs).

## `simplified:authorization_expires`

The date and time (if known) when the user's authorization will expire (e.g. because their library card expires). This value MUST be a time in "YY-MM-DDTHH:MM:SSZ" format (with the time being UTC).

## `simplified:fines`

A JSON object. The `value` key of the object corresponds to a string representing the amount of money this patron owes in library fines. The `currency` key corresponds to the currency in which the fines are owed. This MUST be a 3-letter ISO 4217 currency code, e.g. "USD".

## `simplified:synchronize_annotations`

A boolean value. If this is set to `true`, it indicates that the user wants their client to automatically synchronize local annotations with the [[Web Annotation Protocol|https://www.w3.org/TR/annotation-protocol/]] endpoints that the server thinks are appropriate. If this is set to `false`, the user does not want their e-reader client to automatically synchronize local annotations with those endpoints.

An authentication domain that provides both the Web Annotation Protocol and the User Profile Management Protocol MAY use this as a way for users to opt their clients in or out of WAP. The meaning of `simplified:synchronize_annotations` is undefined unless an authentication domain provides _both_ the User Profile Management Protocol and the Web Annotation Protocol.

Even if this is set to `false`, the client may synchronize local annotations with some _other_ Web Annotation Protocol server, if the user has directed it to do so.