The Library Simplified project aims to streamline the experience of borrowing electronic books from a public library. We offer:

* A mobile e-reader (iOS and Android) that focuses on patron experience and accessibility.
* Seamless integration of collections from multiple vendors.
* Easy access to a wide variety of public domain and open-access books.

**Our goal is to make borrowing ebooks as easy as buying them.**

This wiki is written by the Library Simplified developers and intended for use by technical staff at public libraries. For details on how to get your library involved, see [[the Library Simplified website|http://www.librarysimplified.org/]].

# Overview

Library Simplified is a collection of middleware, server software and mobile client applications for iOS and Android that libraries may use to deliver digital content to their patrons for experimentation or daily eBook services. It is designed to provide a unique user experience for the lending of eBooks and other digital content. It integrates with technologies and systems common to libraries like:

* A Integrated Library System/Library Management System (ILS/LMS)
* Online Public Access Catalogue (OPAC)
* Single Sign On (SSO) credential management system 
* Commercial eBook hosting and distribution services

Currently, our active public repositories include:

- **Client-side Readers**
  - [iOS app](https://github.com/NYPL-Simplified/Simplified-iOS)
  - [Android app](https://github.com/NYPL-Simplified/Simplified-Android)

- **Backend Servers**
  - [Circulation Manager](https://github.com/NYPL-Simplified/circulation)
  - [Open Access Content Server](https://github.com/NYPL-Simplified/content-server)
  - [Metadata Wrangler](https://github.com/NYPL-Simplified/metadata-wrangler)
  - [Server Core](https://github.com/NYPL/Simplified-server-core)
  - [Data Store](https://github.com/NYPL-Simplified/data)

# Installation and setup

* [[Deployment Instructions]]
* [[Automated Jobs and how to use them|AutomatedJobs]]
* [[The configuration file|Configuration]]
* [[Setting up lanes|LaneConfiguration]]
* [[Error handling|ErrorHandling]]
* [[Contributing to the project|Contributing]]

# Architecture

* [[Overview|ArchitecturalOverview]]
* [[The standards we use|StandardsDocuments]]
* [[The generic metadata layer|GenericMetadata]]
* [[The generic circulation API|GenericCirculation]]
* [[The annotation API|Annotations]]

# Integrations

## Sources of content

## Primarily ebooks

* [[The Library Simplified open-access content server|OpenAccessContentServer]]
* [[Overdrive]]
* [[Bibliotheca|Bibliotheca]] (formerly 3M Cloud Library)
* [[Axis 360|Axis360]]
* [[OneClickDigital|One Click Digital]] (from Recorded Books)
* Many content sources [[have no API|ContentSourcesWithoutAPI]] and cannot be properly integrated into Simplified.

## ILSes and other patron authenticators

* [[SIP]] - A common protocol for authenticating patrons
* In addition to SIP, Sierra offers a number of APIs:
 - The [[Millenium]] Patron API
 - The [[Sierra]] REST API
 - The SOAP Patron API
* [[SirsiDynix]]
* [[Evergreen]]
* [[Koha]]
* [[Clever]]
* [[FirstBook]]


## Sources of open-access books

* [[Project Gutenberg|ProjectGutenberg]]
* [[unglue.it|UnglueIt]]
* [[Standard Ebooks|Standard Ebooks]]
* [[Others|OpenAccessContentSources]]

## Sources of metadata

* [[VIAF]]
* [[OCLC Linked Data]]
* [[OCLC Lookup]]
* [[Others|MetadataSources]]

## Other integrations

* [[DRM access control servers|DRM]]

# Metadata

* [[Classification]]
* [[Discovery]]