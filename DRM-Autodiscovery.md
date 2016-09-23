The Library Simplified circulation manager and its mobile clients are designed to negotiate access to content encrypted by various DRM schemes, without requiring any special action on the part of a patron who just wants to read a book.

DRM systems are generally designed for the use case of a book distributor (e.g. a bookstore) providing a white-label app to its customers. This assumption does not hold for Library Simplified. We have an ecosystem in which multiple apps can access multiple libraries, and each library might grant access to books hosted by multiple distributors.

DRM specs leave certain parts of the control flow undefined, especially as regards the initial setup of a patron's DRM account. They assume the bookstore will come up with something to bridge this gap and hard-code it into its white-label app.

We can't just hard-code something into an app, but the ecosystem won't work if every circulation manager is allowed to come up with its own way of bridging this gap. This page tracks the additional conventions we've come up with to fill in the gaps.

## Adobe ACS

When you borrow a book that is held in an ACS server, you are given an ACSM file. (Media type: `vnd.adobe/adept+xml`) This is a short XML document describing your rights under the license. If you have an ACSM file _and_ an Adobe ID, you can put both of them into a black box of Adobe code, turn the crank, and get both the actual book and a key that lets you decrypt the book.

The good thing about Adobe IDs is that once you have one, you can use it for _any_ ACSM file from any source. The bad thing is that it's relatively difficult to get an Adobe ID. You have to get it from a Vendor ID server. Running a Vendor ID server costs big money, and a Vendor ID server is only supposed to serve clients of one specific app.

If a circulation manager is not associated with any Vendor ID server, there's nothing we can do but hope the client already has an Adobe ID from some other source. If a circulation manager _is_ associated with a Vendor ID server, there's a simple way to tell someone who has just checked out a book that they can easily get a Vendor ID if they need one. (Most circulation managers will be associated with the Open Ebooks Vendor ID server, so we should be able to do this.)

The details are covered in the [Vendor ID Service](https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit#) spec. I've slightly modified the tag names for this example. Here's the OPDS entry you might get back after borrowing a book.

```
<entry>
<link rel="http://opds-spec.org/acquisition" href="..." type="vnd.adobe/adept+xml">
  <opds:indirectAcquisition type="application/epub+zip"/>
  <drm:drm type="http://librarysimplified.org/terms/drm/ACS">
    <drm:vendor>Open Ebooks</acs:vendor>
    <drm:client-token>eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwOi8vd3d3
LmxpYnJhcnlzaW1wbGlmaWVkLm9yZy8iLCJzdWIiOiIxMjM0NTY3OCJ9.DTKf7eva3YBb7RzMWs_5EK36wQPfk_RMxBf7UvLAgxc</drm:client-token>
  </drm:drm>
</link>
</entry>
```

The `<link>` tag explains how to get an ACSM file.

The `<indirectAcquisition>` tag inside the `<link>` tag says that you'll be able to exchange the ACSM file for an EPUB. But the ACSM file isn't enough on its own; you also need an Adobe ID.

The `<drm>` tag inside the `<link>` tag tells you how to get an Adobe ID if you don't already have one. The `<client-token>` is used by the ACS client as the `authData` argument when calling `dpdrm::DRMProcessor::initSignInWorkflow`. (details are in Adobe's Vendor ID Specification).

Where does that `<client-token>` value come from? It's an encoded JSON Web Token that is calculated by the circulation manager. This value can be calculated on the fly given only the patron identifier. Since we just loaned out a book (or we are looking at a specific patron's loans), the patron identifier has already been loaded from the database. This means calculating a new token for every request doesn't put any undue burden on the circulation manager.

An OPDS feed that has multiple `<link>` tags to an ACS-encrypted resource should provide an identical `<drm>` tag for each one. The circulation manager should not omit the `<drm>` tag because it believes the patron already has an Adobe ID; the patron might be using a new device, and this procedure will allow the patron to look up an existing Adobe ID.

### Failure modes

An Adobe ID can have at most six device IDs associated with it. An attempt to register a seventh device will fail. Unfortunately edge cases mean patrons frequently reach the device limit with one or two devices.

We need a way to clear all the device IDs from an existing Adobe ID. If all else fails, a client needs a way to wipe out its Adobe ID and get a brand new one.

## Sony URMS

### Failure modes

## LCP

## Failure modes