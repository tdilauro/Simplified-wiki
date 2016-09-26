# Document formats

## OPDS (Open Publication Distribution System)

OPDS is an XML document format based on Atom. We use it to talk about library books and give instructions for borrowing them. OPDS is at the core of our strategy: we use it to communicate with patrons as well as for machine-to-machine integrations.

## Authentication for OPDS

Authentication for OPDS is a JSON document format used by libraries and bookstores to guide users through the authentication process, and add light branding to a generic OPDS reader.

[[The specification draft|https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw]]

## ODL (Open Distribution to Libraries)

ODL is a JSON document format used by ebook distributors to send licensing information to libraries.

[[The specification draft|https://docs.google.com/document/d/1jaHZSsMCZY80rH5sqJcp_kya0W2Dtkda7MrdFBkrOg8/edit]]

# DRM systems

## ACS (Adobe Content Server)

A proprietary DRM system commonly used by library ebook distributors. Access to encrypted books and decryption keys is controlled by ACSM files (media type `vnd.adobe/adept+xml`).

Documentation is confidential.

### Adobe Vendor ID

An API for turning library-specific authentication credentials into Adobe IDs that can be used to fulfill ACSM files.

Documentation is confidential.

### Vendor ID Service

[An open specification](https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit#heading=h.jxwemo85jady) for an Adobe Vendor ID service that can be used by every library participating in a multi-library application.

## URMS (Sony User Rights Management Solution)

A proprietary DRM system. Access to encrypted books is unrestricted; access to decryption keys requires registering a client with a 'store' to create a 'profile', then passing in a short string called a CCID.

Documentation is confidential.

## LCP (Licensed Content Protection)

An open standard for DRM.

[The spec](https://docs.google.com/document/d/1oNfXQZRSGqwpexLrhw0-0a2taEvVDXS9cPs8oBKLb0U/edit#).

A [License Status Document](https://docs.google.com/document/d/1ErBf0Gl32jNH-QVKWpGPfZDMWMeQP7dH9YY5g7agguQ/edit#) describes the rights granted to a reader by a license.

# Protocols

# Vocabularies

## Dublin Core

## Bibframe

## schema.org

## librarysimplified.org

# Miscellaneous

* [JSON Web Token](https://tools.ietf.org/html/rfc7519)
* [Problem detail document](https://tools.ietf.org/html/rfc7807)