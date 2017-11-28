There are three front-end web applications for the Simplified project:
- [opds-web-client](https://github.com/NYPL-Simplified/opds-web-client)
  - A generic catalog browser that can display any OPDS feed, and does not handle any circulation manager-specific OPDS extensions. The other applications build on this.
  - Demo: http://opds-browser-demo.herokuapp.com
- [circulation-patron-web](https://github.com/NYPL-Simplified/circulation-patron-web)
  - A patron-facing catalog app specifically for circulation managers. It does include circulation manager-specific things, and it can be themed differently for different libraries.
  - This is currently only used in production by Open eBooks (https://catalog.openebooks.us). NYPL has a QA version.
- [circulation-web](https://github.com/NYPL-Simplified/circulation-web)
  - The administrative interface for the circulation manager. This is the primary interface for system administrators to set up circulation manager settings and configure integrations, and it's used by every library that's deploying the circulation manager. It can also be used by libraries to edit book metadata and create lists and lanes.
  - This is closely coupled to the [circulation manager](https://github.com/NYPL-Simplified/circulation) and requires a circulation manager to run. It will be served at "/admin".