The Library Simplified circulation manager and its mobile clients are designed to negotiate access to content encrypted by various DRM schemes, without requiring any special action on the part of a patron who just wants to read a book.

DRM systems are generally designed for the use case of a book distributor (e.g. a bookstore) providing a white-label app to its customers. This assumption does not hold for Library Simplified. We have an ecosystem in which multiple apps can access multiple libraries, and each library might grant access to books hosted by multiple distributors.

DRM specs leave certain parts of the control flow undefined, especially as regards the initial setup of a patron's DRM account. They assume the bookstore will come up with something and hard-code it into its white-label app.

We can't just hard-code something into an app, but the ecosystem won't work if every circulation manager is allowed to come up with its own way of bridging this gap. This page tracks the additional conventions we've come up with to fill in the gaps.

## Adobe ACS

### Failure modes

## Sony URMS

### Failure modes

## LCP

## Failure modes