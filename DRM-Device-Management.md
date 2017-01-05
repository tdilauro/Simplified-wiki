# Summary

Library patrons frequently find themselves exceeding the device limits
set by DRM schemes, even though very few people actually have more
devices than the limit allows. The most common culprit is redundant
device IDs created by multiple activations of a single device. In
general, without a record of previously used device IDs, there is no
way to deactivate IDs that are no longer in use.

This document defines the DRM Device Management Protocol, which allows
a patron to maintain a list of the device IDs associated with their
DRM account ID, on the same server that issued that account ID. If a
patron exceeds the device limit, they can retrieve a list of their
device IDs and clear out the old ones.

The deactivation of old device IDs is an intermediate step between
deactivating a single device ID (the normal use case, which is
supposed to cover everything, but obviously doesn't) and invalidating
the DRM account ID itself (an emergency procedure which resets the
device limit but also invalidates the patron's current loans).

Deactivating a device that's still in use is inconvenient but
ultimately harmless. The device will automatically be reactivated the
next time its owner tries to open a book on that device.

Although in theory this protocol is generic, it was designed for use with the ACS DRM scheme only. The behavior of this protocol in terms of other DRM schemes is undefined.

# Who should implement?

Any library that serves its patrons content encrypted with Adobe ACS DRM and licensed should implement this protocol.
The Library Simplified circulation manager implements this protocol, so most SimplyE participating libraries don't have to do anything.

A library may choose not to implement this protocol. SimplyE will continue to work, but there will be more bogus incidents where patrons exceed their device limit.

# Privacy implications

The proposal is that we track the association of a patron's DRM account ID with their device IDs. We have weighed the risks against the benefit to patrons and decided to proceed. Our reasoning:

* The risk is not of a new type. The DRM vendor server that hands out the device IDs also knows both the device IDs and the DRM account ID.
* ACS device IDs are opaque strings without any semantic content. You cannot use a device ID to determine anything about a device. You also cannot use it to prove ownership of a particular device, unless you also have access to that device. The only information you can find by looking at a list of ACS device IDs is _how many_ devices a person has used.

# The `http://librarysimplified.org/terms/drm/rel/devices` link relation

We introduce the link relation
`http://librarysimplified.org/terms/drm/rel/devices`, which
indicates an endpoint into the DRM Device ID Management Protocol.

```
<link rel="http://librarysimplified.org/terms/drm/rel/devices"
      href="https://circulation.librarysimplified.org/AdobeAuth/devices">
```

A library that wants to provide this link should include it in a
`<drm:licensor>` tag. Example:


```
<feed>
  <drm:licensor drm:vendor="VENDOR" drm:scheme="http://librarysimplified.org/terms/drm/scheme/ACS">
    <drm:clientToken>sometoken</drm:clientToken>
    <link rel="http://librarysimplified.org/terms/drm/rel/devices"
          href="https://vendor-id.server/AdobeAuth/devices">
  </drm:licensor>
</feed>
```

If the link is not present, there is no expectation that the patron
will be able to keep track of their device IDs for that DRM licensor.

# The `vnd.librarysimplified/drm-device-id-list` media type

The media type `vnd.librarysimplified/drm-device-id-list` is a
newline-delimited text format similar to `text/uri-list`. Example:

```
10934-234fasd-45893we
89150-ztoi4j-543981jg
```

Each line in the document is a known DRM device ID.

# The protocol

## Authentication

All requests to a library's DRM Device ID Management Protocol endpoint are authenticated using the same technique used to get that library's OPDS bookshelf feed.

## GET

Sending GET to the DRM Device ID Management Protocol endpoint yields a
document of media type
`vnd.librarysimplified/acs-device-id-list`. This list contains all
known device IDs associated with the authenticated DRM account ID.

A [[Link-Template header|https://tools.ietf.org/html/draft-nottingham-link-template-01]]
with the 'item' relationship is also served. This link template
explains how to construct the ID for a specific device ID within the
protocol. The `{id}` variable is defined to stand for the device ID.

An example response:

```
200 OK
Content-Type: vnd.librarysimplified/drm-device-id-list
Link-Template: <https://vendor-id.server/AdobeAuth/devices/{id}>; rel="item"

10934-234fasd-45893we
89150-ztoi4j-543981jg
```

From this you can figure out that there are two known device IDs, and
the URL for dealing with the device ID "89150-ztoi4j-543981jg" is
`https://vendor-id.server/AdobeAuth/devices/89150-ztoi4j-543981jg`.

## POST

Sending POST to the endpoint is a request to register one or more
device IDs. The request entity-body should be a document of media type
`vnd.librarysimplified/drm-device-id-list`.

## DELETE

In the ACS case, a client sends DELETE to a device ID's URL (generated from the `rel="item"` link template) to signal to the server
that the client has deactivated that device ID, or knows for a fact
that that device ID is no longer active. In the ACS case, a DELETE request is _not_ a
request for the server to actually deactivate the device ID. The server has no such power, and only the client can deactivate a device ID, by communicating with Adobe.

# New considerations

Since one device can revoke another device's activation, there will be
a new state where a SimplyE user is logged in to a library but has no
valid device registration (because another device revoked it). The
SimplyE client should detect this condition and attempt to re-register
before raising an error.

