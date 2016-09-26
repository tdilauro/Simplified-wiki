The Library Simplified circulation manager and its mobile clients are designed to negotiate access to content encrypted by various DRM schemes, without requiring any special action on the part of a patron who just wants to read a book.

DRM systems are generally designed for the use case of a book distributor (e.g. a bookstore) providing a white-label app to its customers. This assumption does not hold for Library Simplified. We have an ecosystem in which multiple apps can access multiple libraries, and each library might grant access to books hosted by multiple distributors.

DRM specs leave certain parts of the control flow undefined, especially as regards the initial setup of a patron's DRM account. They assume the bookstore will come up with something to bridge this gap and hard-code it into its white-label app.

We can't just hard-code something into an app, but the ecosystem won't work if every circulation manager is allowed to come up with its own way of bridging this gap. This page tracks the additional conventions we've come up with to fill in the gaps.

## Adobe ACS

When you borrow a book that is held in an ACS server, you are given an ACSM file. (Media type: `vnd.adobe/adept+xml`) This is a short XML document describing your rights under the license. If you have an ACSM file _and_ an Adobe ID, you can put both of them into a black box of Adobe code, turn the crank, and get both the actual book and a key that lets you decrypt the book.

The good thing about Adobe IDs is that once you have one, you can use it for _any_ ACSM file from any source. The bad thing is that it's relatively difficult to get an Adobe ID. You have to get it from a Vendor ID server. Running a Vendor ID server costs big money, and a Vendor ID server is only supposed to serve clients of one specific app.

If a circulation manager is not associated with any Vendor ID server, there's nothing we can do but hope the client already has an Adobe ID from some other source. If a circulation manager _is_ associated with a Vendor ID server, there's a simple way to tell someone who has just checked out a book that they can easily get a Vendor ID if they need one. (Most circulation managers will be associated with the Open Ebooks Vendor ID server, so we should be able to do this.)

The details are covered in the  spec. I've slightly modified the tag names for this example. Here's the OPDS entry you might get back after borrowing a book.

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

Where does that `<client-token>` value come from? In this case, it's an encoded JSON Web Token that is calculated by the circulation manager according to the rules in the [Vendor ID Service](https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit#) spec. But it can be any string that meets the criteria laid out in Adobe's Vendor ID spec. Since it shows up a lot but is rarely used, it should be a value that can be calculated very quickly on the fly, to avoid undue burden on the circulation manager.

An OPDS feed that has multiple `<link>` tags to an ACS-encrypted resource should provide an identical `<drm>` tag for each one. The circulation manager should not omit the `<drm>` tag because it believes the patron already has an Adobe ID; the patron might be using a new device, and this procedure will allow the patron to look up an existing Adobe ID.

### Failure modes

An Adobe ID can have at most six device IDs associated with it. An attempt to register a seventh device will fail. Unfortunately edge cases mean patrons frequently reach the device limit with one or two devices.

To avoid this, a server may offer an authenticated client a way to get a list of their authenticated device IDs. One registered device may deregister a different registered device if it knows the ID. Since reregistration happens automatically under this system, a device that bumps into the device limit can deregister another device and register itself automatically.

If all else fails, a client needs a way to wipe out its Adobe ID and get a brand new one. This will mean the patron loses access to previously borrowed books.

The mechanisms for registering a device ID with the Vendor ID service, getting a list of registered device IDs, and wiping out an Adobe ID are not defined here.

## Sony URMS

URMS adds a number of troubling complications to the DRM story.

1. A book in ACS is associated with a user when the ACSM file is fulfilled, but a book in URMS is licensed to a user from the beginning. Decrypting a URMS book requires a CCID (an ID unique to a specific encrypted file) and a "profile". A profile is the combination of a user ID and device ID.
2. A single Adobe ID can fulfill an ACSM file from any source, but a user needs to set up a separate profile with _every_ source of books.
3. Sony does not define how the license source communicates the CCID to the client. There is no equivalent of an ACSM file.
4. When setting up a profile, the equivalent of Adobe Vendor ID's `authData` string is not generated by the circulation manager; it comes from Sony. For performance reasons, it's not wise to automatically ask Sony for that string every time just in case the patron needs to create a profile. (Especially since a single OPDS feed might need several different versions of this string, to deal with content from different sources.) We need a way for the client to _ask_ for this string if it's necessary.
5. Although the server may tell the client where to download the encrypted document, this is an optional performance improvement. It's possible to download and decrypt a book with nothing but a CCID and a profile. 

Here's how we choose to resolve these issues. 

Before a book is borrowed, this is how we indicate that decrypting a book will require knowledge of URMS:

```
<entry>
 <title>A URMS Book</title>
 <link rel="borrow" href="...">
    <opds:indirectAcquisition
          type="application/atom+xml;type=entry;profile=opds-catalog">
      <opds:indirectAcquisition
            type="vnd.librarysimplified/drm-encrypted;method=urms;decrypts-to=application/epub">
        <opds:indirectAcquisition type="application/epub"/>
      </opds:indirectAcquisition>
    </opds:indirectAcquisition>
  </opds:indirectAcquisition>
 </link>
</entry>
```

This says:

1. Borrow the book and you will get an OPDS entry.
2. The OPDS entry will help you negotiate the URMS API.
3. Once you make it through the API you will get an EPUB.

Once you borrow the book you will be served an OPDS entry like this:

```
<entry>
 <link rel="acquisition" href="urn:urms-ccid:KDFASDJFLIAKSJ" type="vnd.librarysimplified/drm-encrypted;method=urms;decrypts-to=application/epub">
  <opds:indirectAcquisition type="application/epub"/>
  <link rel="acquisition" href="https://host/foo.epub" type="vnd.librarysimplified/drm-encrypted;method=urms;decrypts-to=application/epub" />
 </link>
 <link rel="http://opds-spec.org/drm/register-client" 
       href="https://host/register/CEL" drm:storeID="CEL"
       drm:type="urms"/>
</entry>
```

This says:

1. You can get the book from `http://host/foo.epub`, but it's going to be encrypted with URMS, and you won't be able to turn it into a usable form.
2. To get the book in `application/epub`, you need to start from the CCID, which is `KDFASDJFLIAKSJ`.
3. The URMS Store that provides the book is `CEL`.
4. If you need to create a profile with `CEL`, you can send an authenticated GET request to `https://host/register/CEL`.

It's nice to have that direct link to the EPUB, because it means you can start downloading the book while you wait to fulfill the loan, but it's not mandatory. If the EPUB link is missing, you can still fulfill the book with just a CCID and a URMS profile.

### Client registration

When the client makes a GET request to the `/drm/register-client` link (in this case, `https://host/register/CEL`), it needs to send whatever authentication credentials the circulation manager expects, the same as if it were following an OPDS `bookshelf` link.

Behind the scenes, the circulation manager will make sure the patron has a registered user account with the appropriate URMS Store. Then it will request an auth token for that patron from the URMS Store. The auth token will be passed back to the client (it looks like `3375:z4nk82tdj32hf4ad`). The client can use this token to create a profile that connects the patron to their device.

### Failure modes

## LCP

Before a book is borrowed, here's how we indicate that reading a book will require knowledge of LCP.

```
<entry>
 <title>An LCP Book</title>
 <link rel="borrow" href="...">
    <opds:indirectAcquisition
          type="application/atom+xml;type=entry;profile=opds-catalog">
      <opds:indirectAcquisition
            type="application/vnd.readium.lcp.license-1.0+json">
        <opds:indirectAcquisition type="application/epub"/>
      </opds:indirectAcquisition>
    </opds:indirectAcquisition>
  </opds:indirectAcquisition>
 </link>
</entry>
```

This says:

1. Borrow the book and you'll get an OPDS entry.
2. That entry will contain a link to an LCP License Document
3. You'll be able to use the LCP License Document to actually get the book.

Once you borrow the book, you'll get an OPDS entry that looks like this:

```
<entry>
 <title>An LCP Book</title>
 <link rel="acquisition" href="...">
    <opds:indirectAcquisition
          type="application/vnd.readium.lcp.license-1.0+json">
      <opds:indirectAcquisition type="application/epub"/>
    </opds:indirectAcquisition>
    <drm:drm type="http://librarysimplified.org/terms/drm/LCP">
      <drm:client-token>sodih43oth489</drm:client-token>
    </drm:drm>
  </opds:indirectAcquisition>
 </link>
</entry>
```

The `drm:client-token`, "sodih43oth489" in this example, is the LCP user key. A client can use this to get access to the book and decrypt it. The provider may choose to generate a different user key for each loan, or to use the user key as described in Section 4 of the LCP specification: "the result of applying a hashing function on the User Passphrase", a string of text known by the user. 

If the provider defines the user key as defined in Section 4, DRM autodiscovery beyond this point is limited to making sure the patron can come up with the User Passphrase given the prompt. If the provider generates a different user key for each loan, it MUST provide that key as `drm:client-token` here, because there is no User Passphrase.

If the provider serves `drm:client-token` upon initial checkout, it MUST also serve the same `drm:client-token` every time it describes the loan (e.g. when listing books on the patron's bookshelf). This way, a patron will always be able to bring a book onto a fresh device and read it there.

In a library setting, the LCP user key SHOULD be a different value for every loan. This suggestion takes precedence over the statement in 4.4 of the LCP spec that "the Provider should use the same User Key for all licenses issued to the same User." This allows a library to evade a major privacy problem: the existence of a persistent identifier (such as the Adobe ID or URMS client ID) associated with every one of a patron's loans and tracked outside the library's control.

The downside of providing a different user key for every loan is that the patron will be unable to bring their book into an e-reader application that does not also support OPDS and this DRM autodiscovery protocol.

## Failure modes

LCP does not have a notion of a client ID as distinct from the User Key, and does not impose limits on the number of devices that can be used to fulfill a loan. As such it is not subject to the common failure modes encountered by library patrons.

## Work to be done

* Formally define the `vnd.librarysimplified/drm-encrypted` media type, with its `method` and `decrypts-to` parameters.
* Formally define the extension namespace for DRM with its `drm`, `vendor`, `client-token`, `type`, and `clientID` tags and attributes.
* Define a DRM-independent (?) mechanism for retrieving lists of registered device IDs.
* Define a DRM-independent (?) mechanism for completely resetting a client ID.
