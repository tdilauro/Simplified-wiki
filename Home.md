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

- **[[Front-end Web Applications|Simplified-Front-end-Web-Apps]]**
  - [Patron Facing Client](https://github.com/NYPL-Simplified/circulation-patron-web)
  - [Circulation Manager Admin Interface](https://github.com/NYPL-Simplified/circulation-web)

- **Deployment Tools**
  - [Circulation Manager Docker configuration](https://github.com/NYPL-Simplified/circulation-docker)

# Installation, Configuration, and Development

* [[Getting Your Books Into SimplyE|GettingYourBooksIntoSimplyE]]
* [[Connecting the Library Simplified circulation manager to your ILS|AuthenticationSetup]]
* [[The configuration file|Configuration]]
* [[Setting up lanes|LaneConfiguration]]
* [[Contributing to the project|Contributing]]
* [[Deployment for development|Deployment Instructions]]
* [[Creating new server-side integrations|NewIntegrations]]

# Deployment and Docker
  - [[Deployment: Nginx & uWSGI|Deployment:-Nginx-&-uWSGI]]
  - [[Deployment: Quickstart with Docker|Deployment:-Quickstart-with-Docker]]
  - [[Automated Jobs and how to use them|AutomatedJobs]]
  - **Docker AWS examples:**<br />
    These are rough guides detailing a few tested methods for deploying the Library Simplified Circulation Manager on AWS. They are by no means complete, and we highly recommend independent research to identify AWS platforms and orchestrations that best suit your needs.
    - [[nypl/circ-webapp on Elastic Beanstalk|Docker-Deployment-Example:-Elastic-Beanstalk-with-CloudWatch]]
    - [[nypl/circ-scripts on ECS|Docker-Deployment-Example:-ECS-scripts-container]]

# Troubleshooting
  - [[Error handling|ErrorHandling]]
  - [[Troubleshooting "books don't show up in the feed" problems|BooksDontShowUp]]

# Architecture

* [[Overview|ArchitecturalOverview]]
* [[Supporting multiple libraries on one server|MultiTenancy]]
* [[The standards we use|StandardsDocuments]]
* [[The generic metadata layer|MetadataLayer]]
* [[The generic circulation API|GenericCirculation]]
* [[The generic authentication API|GenericAuthentication]]
* [[The annotation API|Annotations]]
* [[The Simple Signup Protocol|Simple-Signup-Protocol]]
* [[DRM autodiscovery|DRM-Autodiscovery]]
* [[Short Client Token|Short-Client-Token]] for federated patron management
* [[The ACS Device Management Protocol|DRM-Device-Management]]
* [[OPDS For Distributors|OPDSForDistributors]]
* [[The User Profile Management Protocol|User-Profile-Management-Protocol]]
* [[OPDS Directory Registration Protocol|OPDS-Directory-Registration-Protocol]]
* [[Temporary Token Access Control|TemporaryTokenAccessControl]]
* [[Our extensions|Authentication-For-OPDS-Extensions]] to the [[Authentication for OPDS spec|https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw/edit#heading=h.r2fysm93j6kk]]
* [[APIs implemented by the Metadata Wrangler|Metadata-Wrangler]]

# Proposed changes

* [[Wikiregistry of unique library IDs|UniqueLibraryIDs]]
* [[Integrating the circulation manager with the library registry|CirculationManagerToLibraryRegistry]]
* [[Improvements to contributor tracking|ContributorImprovements]]
* [[RFC: Canonical Contributors|RFC:-Canonical-Contributors]]

# Recent changes
* [[Multiple collections in one circulation manager|MultipleCollections]]
* [[Multiple libraries in one circulation manager|MultipleLibraries]]
* [[Multiple authentication mechanisms in one circulation manager|MultipleAuthenticationMechanisms]]
* [[Library registry search algorithm|LibraryRegistrySearchAlgorithm]]
* [[Library registry|LibraryRegistry]]
* [[Storing lane configuration in the database|DatabaseLanes]]

# Integrations

## Sources of content

## Primarily ebooks

* [[The Library Simplified open-access content server|OpenAccessContentServer]]
* [[Overdrive]]
* [[Bibliotheca|Bibliotheca]] (formerly 3M Cloud Library)
* [[Axis 360|Axis360]]
* [[OneClickDigital|One Click Digital]] (from Recorded Books)
* [[desLibris]] (from the Canadian Electronic Library)
* Many content sources [[have no API|ContentSourcesWithoutAPI]] and cannot be properly integrated into Simplified.

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
* [[Adobe Vendor ID server|AdobeVendorID]]

# Metadata

* [[Classification]]
* [[Discovery]]