The Library Simplified circulation manager and its mobile clients are designed to negotiate access to content encrypted by various DRM schemes, without requiring any special action on the part of a patron who just wants to read a book.

DRM systems are generally designed for the use case of a book distributor (e.g. a bookstore) providing a white-label app to its customers. This assumption does not hold for Library Simplified. We have an ecosystem in which multiple apps can access multiple libraries, and each library might grant access to books hosted by multiple distributors.

DRM specs leave certain parts of the control flow undefined, especially as regards the initial setup of a patron's DRM account. They assume the bookstore will come up with something and hard-code it into its white-label app.

We can't just hard-code something into an app, but the ecosystem won't work if every circulation manager is allowed to come up with its own way of bridging this gap. This page tracks the additional conventions we've come up with to fill in the gaps.

## Adobe ACS

When you borrow a book that is held in an ACS server, you are given an ACSM file. (Media type: `vnd.adobe/adept+xml`) This is a short XML document describing your rights under the license. If you have an ACSM file _and_ an Adobe ID, you can put both of them into a black box of Adobe code, turn the crank, and get both the actual book and a key that lets you decrypt the book.

The good thing about Adobe IDs is that once you have one, you can use it for _any_ ACSM file from any source.

### Failure modes

## Sony URMS

### Failure modes

## LCP

## Failure modes