# Summary

The User Settings Management Protocol (hereafter the "Protocol") is a very simple HTTP-based protocol for retrieving and modifying user settings. Although designed for use in public libraries, the scope of this protocol is not limited to them.

## Who should implement?

Any library that wants to participate in the SimplyE program should implement the Protocol. The Library Simplified circulation manager implements the Protocol, so most SimplyE participating libraries don't have to do anything.

A library may choose not to implement the Protocol. SimplyE will work as described in the "Default behavior" section.

Other organizations may implement the Protocol for other types of user accounts, by defining their own keys for the fields.

# The `http://librarysimplified.org/terms/rel/settings` link relation

In the examples to follow, I'll use `http://settings/` as the URL of a resource that implements the Protocol.

We introduce the link relation `http://librarysimplified.org/terms/rel/user-settings`, which indicates that the link target supports the Protocol. This link relation may be used anywhere.

Here's an HTML example of a link into the User Settings Management Protocol:

```
<a href="https://settings/" rel="http://librarysimplified.org/terms/rel/user-settings" type="vnd.librarysimplified/user-settings+json">
 Change your settings
</a>
```

# Behavior under HTTP

This is mostly common-sense HTTP stuff.

## Authentication

Unauthenticated users may not participate in the Protocol. An unauthenticated request to an Protocol endpoint MUST result in a 401 response code with an authentication demand.

In general, a user may only administer their own settings. Users with superuser privileges MAY use the Protocol to administer settings for other users, but this spec does not define how that might work.

## GET

An Protocol server that receives an authenticated GET request SHOULD send a document of media type `vnd.librarysimplified/user-settings+json`. It MAY send some other media type that supports the same essential features.

## PUT

An authenticated PUT request to a Protocol endpoint SHOULD be accompanied by a document of media type `vnd.librarysimplified/user-settings+json`. It MAY be accompanied by a document of some other media type that supports the same essential features. The Protocol server MUST reject a document it does not understand.

Upon receipt, the server MUST do its best to make the underlying user settings reflect the content of the incoming document (see below). If this is not possible, due to a security violation, nonsensical values, conflicting values, or any other reason, the server MUST send a [[problem detail|https://tools.ietf.org/html/rfc7807]] explaining the problem.

# The `vnd.librarysimplified/user-settings+json` media type

A document with the media type `vnd.librarysimplified/user-settings+json` represents the current state of user settings (when sent from the server to the client) or a desired future state of user settings (when sent from the client to the server).

This document has the form of a single JSON object. Semantics are defined for two keys that MAY appear in the JSON object: `readable` and `writable`. Other keys MAY appear in the JSON object, but this specification does not define their meaning.

## `readable`

The value of `readable` MUST be a single JSON object. The keys of this object correspond to the names of user settings, and the values associated with the keys correspond to the current values of those settings.

When a client PUTs a document to a Protocol endpoint, it SHOULD NOT send a document that includes `readable`. If it does sent a document that includes `readable`, the server MUST ignore it.

Semantics for a few keys are defined in 'Settings' below. Other keys may show up in `readable` but this specification does not define their semantics.

## `writable`

The value of `readable` MUST be a single JSON object. The keys of this object correspond to the names of user settings. When the document is received in response to a GET request, the values associated with the keys correspond to the current values of those settings. 

When the document is submitted along with a PUT request, the values associated with these keys correspond to the desired new values of these settings. If a key is not included, it indicates that the client does not wish to change the value of that setting. If a key is mapped to `null`, it indicates that the client wants to set the value of that setting to a null value.

Semantics for a few keys are defined in 'Settings' below. Other keys may show up in `readable` but this specification does not define their semantics.

## Example

This example conveys two pieces of information that the user cannot change (the amount of their fines and the currency in which their fines are measured) and one piece of information that the user can change (whether or not their reading activity is synchronized with the server).

```
{
 "readable": { "simplified:fines": 4.23,
               "simplified:fine_currency": "USD",
             },
 "writable": { "simplified:synchronize_annotations": false }
}
```

## Other notes

This specification does not define a mechanism for conveying the human-readable names or descriptions of settings.

A document MAY NOT include the same key in both `readable` and `writable`.

# Settings

## `simplified:authorization_expires`

Value: The date and time (if known) when the user's authorization will expire (e.g. because their library card expires). This value MUST be either a time in "YY-MM-DDTHH:MM:SSZ" format (with the time being UTC), or a date in "YY-MM-DD" format.

## `simplified:fines`

Value: The amount of money this patron owes in library fines.

## `simplified:fine_currency`

Value: The currency in which the fines are owed. This MUST be an ISO 4217 currency code, e.g. "USD".

## `simplified:synchronize_annotations`

Value: A boolean value. If this is set to `true`, it indicates that the user wants their e-reader client, when connected to this server, to automatically synchronize local annotations with the server's [[Web Annotation Protocol|https://www.w3.org/TR/annotation-protocol/]] endpoint. If this is set to `false`, the user does not want their e-reader client to automatically synchronize local annotations with the server's WAP endpoint.

A server that provides both the Web Annotation Protocol and the User Settings Management Protocol may use this as a way for users to opt their clients in or out of WAP. If a server provides the User Settings Management Protocol, but not the Web Annotation Protocol, then `simplified:synchronize_annotations` has no meaning.

Even if this is set to `false`, the client may synchronize local annotations with some _other_ Web Annotation Protocol server, if the user has directed it to do so.
