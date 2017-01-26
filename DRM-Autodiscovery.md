The Library Simplified circulation manager and its mobile clients are designed to negotiate access to content encrypted by various DRM schemes, without requiring any special action on the part of a patron who just wants to read a book.

DRM systems are generally designed for the use case of a book distributor (e.g. a bookstore) providing a white-label app to its customers. This assumption does not hold for Library Simplified. We have an ecosystem in which multiple apps can access multiple libraries, and each library might grant access to books hosted by multiple distributors.

DRM specs leave certain parts of the control flow undefined, especially as regards the initial setup of a patron's DRM account. They assume the bookstore will come up with something to bridge this gap and hard-code it into its white-label app.

We can't just hard-code something into an app, but the ecosystem won't work if every circulation manager is allowed to come up with its own way of bridging this gap. This page tracks the additional conventions we've come up with to fill in the gaps.

This document explores the various use cases in context. [[The formal specifications that came out of this process are here.|DRMAutodiscoverySpecs]]

## Adobe ACS

When you borrow a book that is held in an ACS server, you are given an ACSM file. (Media type: `vnd.adobe/adept+xml`) This is a short XML document describing your rights under the license. If you have an ACSM file _and_ an Adobe ID, you can put both of them into a black box of Adobe code, turn the crank, and get both the actual book and a key that lets you decrypt the book.

The good thing about Adobe IDs is that once you have one, you can use it for _any_ ACSM file from any source. The bad thing is that it's relatively difficult to get an Adobe ID. You have to get it from a Vendor ID server. Running a Vendor ID server costs big money, and a Vendor ID server is only supposed to serve clients of one specific app.

If a circulation manager is not associated with any Vendor ID server, there's nothing we can do but hope the client already has an Adobe ID from some other source. If a circulation manager _is_ associated with a Vendor ID server, there's a simple way to tell someone who has just checked out a book that they can easily get a Vendor ID if they need one. (Most circulation managers will be associated with the Open Ebooks Vendor ID server, so we should be able to do this.)

Here's an OPDS entry you might get back after borrowing a book.

```
<entry>
<link rel="http://opds-spec.org/acquisition" href="..." type="vnd.adobe/adept+xml">
  <drm:licensor drm:vendor="NYPL">
    <drm:clientToken>eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwOi8vd3d3
LmxpYnJhcnlzaW1wbGlmaWVkLm9yZy8iLCJzdWIiOiIxMjM0NTY3OCJ9.DTKf7eva3YBb7RzMWs_5EK36wQPfk_RMxBf7UvLAgxc</drm:clientToken>
  </drm:licensor>
  <opds:indirectAcquisition type="application/epub+zip"/>
</link>
</entry>
```

The `<link>` tag explains how to get an ACSM file.

The `<indirectAcquisition>` tag inside the `<link>` tag says that you'll be able to exchange the ACSM file for an EPUB. But the ACSM file isn't enough on its own; you also need an Adobe ID.

The `<licensor>` tag inside the `<link>` tag tells you how to get an Adobe ID if you don't already have one. The `<clientToken>` is used by the ACS client as the `authData` argument when calling `dpdrm::DRMProcessor::initSignInWorkflow`. (details are in Adobe's Vendor ID Specification). The `<vendor>` is used as the `authority` argument.

Where does that `<clientToken>` value come from? In this case, it's an encoded JSON Web Token that is calculated by the circulation manager according to the rules in the [Vendor ID Service](https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit#) spec. But it can be any string that meets the criteria laid out in Adobe's Vendor ID spec. Since it shows up a lot but is rarely used, it should be a value that can be calculated very quickly on the fly, to avoid undue burden on the circulation manager.

If a site's client token is expensive to calculate, it can be kept behind a link, so that it's only calculated when a client requests it:

```
<entry>
<link rel="http://opds-spec.org/acquisition" href="..." type="vnd.adobe/adept+xml">
  <drm:licensor>
    <drm:clientToken drm:href="https://my-site/getAuthData"/>
  </drm:drm>
  <opds:indirectAcquisition type="application/epub+zip"/>
</link>
</entry>
```

The details of how to turn the link into a client token are covered in the spec ["DRM Extensions to OPDS"](https://github.com/NYPL-Simplified/Simplified/wiki/DRMAutodiscoverySpecs#drm-extensions-to-opds).

An OPDS feed that has multiple `<link>` tags to ACS-encrypted resources SHOULD provide an identical `<licensor>` tag for each one. The circulation manager SHOULD NOT omit the `<licensor>` tag because it believes the patron already has an Adobe ID; the patron might be using a new device, and this procedure will allow the patron to look up an existing Adobe ID. However, the `<licensor>` tag is optional and MAY be omitted.

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
 <link rel="http://opds-spec.org/acquisition/borrow" href="..."
       type="application/atom+xml;type=entry;profile=opds-catalog">
    <opds:indirectAcquisition
          type="vnd.librarysimplified/obfuscated;scheme=http://librarysimplified.org/terms/drm/scheme/URMS">
      <opds:indirectAcquisition type="application/epub+zip"/>
    </opds:indirectAcquisition>
  </opds:indirectAcquisition>
 </link>
</entry>
```

This says:

1. Borrow the book and you will get an OPDS entry.
2. The OPDS entry will help you get a document that is subject to the rules of URMS DRM.
3. Once you make it through the API you will get an EPUB.

Once you borrow the book you will be served an OPDS entry like this:

```
<entry>
 <link rel="http://opds-spec.org/acquisition"
       href="ccid-urms:01234567890"
       type="vnd.librarysimplified/obfuscated;scheme=http://librarysimplified.org/terms/drm/scheme/URMS">
  <opds:indirectAcquisition type="application/epub+zip"/>
  <link rel="http://opds-spec.org/acquisition"
        href="https://host/foo.epub"
        type="vnd.librarysimplified/obfuscated;scheme=http://librarysimplified.org/terms/drm/scheme/URMS;original-type=application/epub+zip"
  />
 </link>
 <drm:licensor>
  <drm:clientToken href="https://host/register/CEL"/>
  <drm:serverToken href="http://urms-12345678.eu-west-1.elb.amazonaws.com" vendor="959"/>
 </drm:licensor>
</entry>
```

This says:

1. You can get the book from `http://host/foo.epub`, but it's going to be encrypted with URMS, and you won't be able to turn it into a usable form.
2. To get the book in `application/epub`, you need to start from the CCID, which is `01234567890`.
3. The URMS Store that provides this book is URMS Store 959, and it can be found at `http://urms-12345678.eu-west-1.elb.amazonaws.com`.
4. If you need to create a profile with that URMS Store, you can get an AuthToken by sending an authenticated GET request to `https://host/register/CEL`. That URL acts as described in the [Client Token Protocol](https://github.com/NYPL-Simplified/Simplified/wiki/DRMAutodiscoverySpecs#the-client-token-protocol).
5. You can get _part_ of the way towards fulfilling this link by downloading the encrypted EPUB from `https://host/foo.epub`, but you won't be able to decrypt it without going through the `ccid-urms` URL. It's nice to have this direct link, because it means you can start downloading the book while you wait to fulfill the loan, but it's not mandatory. If the EPUB link was missing, you could still fulfill the book with just a CCID and a URMS profile.

### Failure modes

Each URMS Store can impose a limit on the number of devices associated with a user ID. This can lead to the same problem we see with Adobe IDs -- patrons who've reached the device limit and don't know how to fix the problem. 

Since the URMS Device API offers "Get Devices" and "Delete Device" endpoints, it may be possible for one device to automatically bump off some other device and register itself, without any new mechanism being created.

## LCP

LCP works more or less like ACS. When you borrow a book you get a License Document (media type: `application/vnd.readium.lcp.license-1.0+json`). You can combine the License Document with the "user key" and a client-side secret to download and decrypt the book. One big advantage of LCP is that there is no separate client registration step.

Before a book is borrowed, here's how we indicate that reading a book will require knowledge of LCP.

```
<entry>
 <title>An LCP Book</title>
 <link rel="http://opds-spec.org/acquisition/borrow" href="..."
       type="application/atom+xml;type=entry;profile=opds-catalog">
    <opds:indirectAcquisition
          type="application/vnd.readium.lcp.license-1.0+json">
      <opds:indirectAcquisition type="application/epub+zip"/>
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
 <link rel="http://opds-spec.org/acquisition" href="..."
       type="application/vnd.readium.lcp.license-1.0+json">
    <drm:licensor">
      <drm:client-token>sodih43oth489</drm:client-token>
    </drm:licensor>
    <opds:indirectAcquisition type="application/epub+zip"/>
 </link>
</entry>
```

The `drm:client-token`, "sodih43oth489" in this example, is the LCP user key. A client can use this to get access to the book and decrypt it. The provider may choose to generate a different user key for each loan, or to use the user key as described in Section 4 of the LCP specification: "the result of applying a hashing function on the User Passphrase", a string of text known by the user. 

If the provider defines the user key as defined in Section 4, DRM autodiscovery beyond this point is limited to making sure the patron can come up with the User Passphrase given the prompt. If the provider generates a different user key for each loan, it MUST provide that key as `drm:client-token` here, because there is no User Passphrase.

## Failure modes

LCP does not have a notion of a client ID as distinct from the User Key, and does not impose limits on the number of devices that can be used to fulfill a loan. As such it is not subject to the common failure modes encountered by library patrons.

## Work to be done

* Define a DRM-independent (?) mechanism for retrieving lists of registered device IDs.
* Define a DRM-independent (?) mechanism for completely resetting a client ID.
