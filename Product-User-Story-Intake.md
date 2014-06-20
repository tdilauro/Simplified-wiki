***

### User can sign into app using existing NYPL Credentials

* https://app.asana.com/0/12956401148094/12956401148102

_SUMMARY DESCRIPTION:_ An existing NYPL user with a valid, active library card can use their existing credentials [BARCODE + PIN] to authenticate to the app, and perform any functions which require an account. 

***

### Get a Library Card/Register I can Borrow eBooks

* https://app.asana.com/0/12956401148094/12956401148106
	
_SUMMARY DESCRIPTION_: Settings View of app allows a user app for a library card by providing the user the ability to provide (Name, DOB, Address, User Name, Password, PIN, email, Phone Number)
#####Confirmation Criteria
* The user can log into other digital properties with their newly established credentials
* Information from the form is stored in the appropriate Identity Management database (SSO)
* An acknowledgment email or SMS is sent to the user after submitting the form (System TBD).
* The user's account is created in the ILS
* The user's account is created in our DRM instance
* The user's account is created in BiblioCommons
* The user's account is created in our eBook Distributor (Overdrive, 3M, Axis 360)

***

### Connect my library account to my App/Don't have enter information again
* https://app.asana.com/0/12956401148094/13026724002726

_SUMMARY DESCRIPTION_: This functionality can range from scanning the library barcode to simply providing the necessary credentials and confirming them with the requisite ILS or Identity Management System
#####Confirmation Criteria
* The App can use III ILS Patron API
* The App can use Polaris ILS API
* The app can use SSO (CAS, OAUTH, Shibboleth, other if required - TBD)

### Search available titles/So I can borrow the title for free

* https://app.asana.com/0/12956401148094/13026724002738

_SUMMARY DESCRIPTION_: This is a basic search function of the eBook Collection.  However, it may eventually link to a broader search of the catalogue regardless of format.
* Can enter a titles and get like title results that are available for download in my needed format
* I can search for titles by multiple facets (title, subject, author, genre) and get like title results that are available for download in my needed format

***

### Get recommendations from Library/So I don't have to search for items
* https://app.asana.com/0/12956401148094/13026724002738

_SUMMARY DESCRIPTION_:	This is the core discovery mechanism for the eBook Collection.  However, it may eventually link to a broader set or combined set of recommendation signals to items in the catalogue regardless of format.
* See available titles recommended by Staff (Physical or Digital)
*  See available ebook (Digital) titles that are recommended, down loadable, and of the right digital format for my device

### [Connect to my 3rd party Reading list/So I can record my reads]
* https://app.asana.com/0/12956401148094/13026724002741
_SUMMARY DESCRIPTION_:Users who are part of online book clubs or actively manage their reading can link the app to those existing venues for recording their reading history or story in their books
#####Confirmation Criteria
* The App can transact on my Google Books Reading List
* The App can transact on my Good Reads books shelf API
* 3rd Party Book List Integration

***

### Create Wish List/So I can get notifications when the book is available 

* https://app.asana.com/0/12956401148094/13026724002752

_SUMMARY DESCRIPTION_:This allows users to create lists of book they may want to borrow from the library
#####Confirmation Criteria
* The App can use my Google Books Reading List to identify wished for books and whether they become a part of the NYPL collection
* The App can use my Good Reads books shelf API to wished for books and whether they become a part of the NYPL collection

### [Be recommended only titles that are available/So I don't have to browse through titles not available to read now]
* https://app.asana.com/0/12956401148094/13026724002754

_SUMMARY DESCRIPTION_:This feature or setting allows discovery of books that can be accessed immediately by the reader with out having to wait in hold queues.
#####Confirmation Criteria
* I get only Titles in our Catalogue/Collection (Not sold by our platform vendor but unlicensed by NYPL)
* I get Titles in the format my device and app supports (If content is only in Kindle, and I am non iPhone don't bother)
* I get only titles that have an available copy 

### [Borrow a physical copy if the eBook isn't available/So I don't have to wait to read the book]

* https://app.asana.com/0/12956401148094/13026724002756

_SUMMARY DESCRIPTION_:Based on the popularity of the searched for title, I may be willing to get an alternate digital version (Audio) or  physical copy.  I would like to see the alternate versions held in the broader collection
#####Confirmation Criteria
* I can place a hold on the physical format if the digital format is not available
* The hold reservation is in the ILS/or digital CMS
* The physical title copy is available for pick-up at branch
* The alternate formats are displayed in the apps title detail page (Assume all searches are for eBooks)

***

### Find and Download the App/So I can try the app

* https://app.asana.com/0/12956401148094/13026724002758

_SUMMARY DESCRIPTION_:This an ability of the app to be rapidly acquired or distributed by users
#####Confirmation Criteria
* The App is In the Apple App Store
* The App is in Google Play (if applicable)
* The App is on the NYPL.org
* The App download location can be shared by a user (email, Social Network)

***

### Use the App on Smart Phone

* https://app.asana.com/0/12956401148094/13058132459368

#####Confirmation Criteria
* The App can be installed on an iPhone
* The App can be installed on an Android
* The App can be installed on Windows Phone

***

### Use the App on my Tablet

* https://app.asana.com/0/12956401148094/13058132459372

* The App appropriately uses the screen real-estate
* The App adjust the views as necessary to not require pixilation
* The Can render content without formatting issues
* The App can be installed on iOS
* The App can be Installed on Android (TBD)
* The App can be Installed on Windows Tablet (TBD)

### Create and account using my email credentials/Don't have enter information again
_SUMMARY DESCRIPTION_:This allows users to sign up for the app using their establish credentials for email or social network like Facebook, Google or Yahoo.  This also allows app to capture data needed for library card registration.
#####Confirmation Criteria
* The App can use Oauth2.0 or OpenID for GMAIL
* The App can use SOAP for MSN (?)

***

### Create an account using my device credentials/Don't have enter information again

* https://app.asana.com/0/12956401148094/13058132459384

_SUMMARY DESCRIPTION_: This allows the user to establish their credentials using the already established credentials on the platform the are using.
#####Confirmation Criteria
* The App can us my iCoud ID (iOS)
* The App can use my SIM ID and or Android Device ID (if applicable)

***

### Homescreen has scrollable Lanes of available material [James]

* https://app.asana.com/0/12956401148094/13058132459388

_SUMMARY DESCRIPTION_: Home View of app is "store" with covers of books available to borrow (or download) now, scrollable from side-to-side sorted by categories.


### Bookmark my pages/So I can return to where I left off reading 
* https://app.asana.com/0/12956401148094/13058132459397 (needs clarity)

### Annotate pages/So I can take notes
* https://app.asana.com/0/12956401148094/13058132459399 (needs clarity)

### Copy snippets of text/So I can keep favorite lines	
* https://app.asana.com/0/12956401148094/13058132459401 (needs clarity)

### Share my reads friends via twitter/So I can let my friends know what I'm reading
* https://app.asana.com/0/12956401148094/13058132459403 (needs clarity)
#####Confirmation Criteria
* The user account can invoke Twitter App
* The user account can use Twitter API
* The app allows the user to enter or preserve their twitter credentials

### [Share my reads friends via Facebook/So I can let my friends know what I'm reading
* https://app.asana.com/0/12956401148094/13058132459407

#####Confirmation Criteria
* The user account can invoke FB App
* The user account can use FB wall API
* The app allows the user to enter or preserve their FB credentials

### Preview a book/So I can see if I really want to read the whole book
* https://app.asana.com/0/12956401148094/13058132459411
#####Confirmation Criteria
* The preview is offered along side download
* The preview is used as opposed to a linked lend if the preview is read

### Get Notification is book is available or wait period changed/So that I can find something else or plan my next read
* https://app.asana.com/0/12956401148094/13058132459415
#####Confirmation Criteria
* The app sends a notifications when the books wait time changes by more than 3 days.
* The app collects the users email/SMS credentials

### Know how long I need to wait, not where I am in line/ So that I can find something else or plan my next read
* https://app.asana.com/0/12956401148094/13058132459413
#####Confirmation Criteria
* The app displays the estimated wait time for wish lists
* The app displays the estimated time for titles recommended
* The app displays the estimated time for titles in search results
### 

### Push Notification of major events to phone / So reader knows they have the option to take action
Summary: For big things, namely hold notifications, a notification can be sent to the users device prompting them to take an action.
#####Confirmation Criteria
* The app displays notifications when an event happens
* The App icon badges update when an event happens
* Push notification is sent to a users device any time they have placed an item on hold
* Push notification enables user to go straight to the part of the app where they can take that action
* If user takes action, push notification/badge is removed
* Until user takes action, or action expires, the notification stays on the device, unless explicitly cleared (i.e. does not go away the first time the user just opens the app post-notification)

### Read PDFs as fixed layout ePubs
* https://app.asana.com/0/12956401148094/13171109685224
_Summary_: Given that Readium's SDK can work with fixed-layout epubs, the client should be able to convert PDFs to Epub3 fixed layout and open them within the reading app.

Confirmation Criteria:
* A PDF can be imported as a readable text in my library
Notes:
* Highly dependent on Readium codebase. Will have to see what it's capacity is to actually do this

### Gale Group databases are available as a content source
* https://app.asana.com/0/12956401148094/13171109685228
_Summary_: Gale Group offers a ton of fixed-layout content (in PDF, which will convert to fixed-layout ePub), which will be available to any NYPL user given a NYPL ID (proxy will allow off-site access). App, when signed in as a user, will enable direct download link access to gale content (and open the door to other PDF based DB's)

Confirmation Criteria:
* User w/ valid NYPL ID (aka can use app) can also search/download/read gale group PDF
* PDF viewable within in-app reader (readium)

Outside Requirements:
* SSO Completion
* EasyProxy

### User can import and read their own epub within the app

_Summary_: Users should be able to import their own unencrypted epubs into the reader app and have it show up  as a readable title on their "My bookshelf""

### User can select the languages they wish to read and be recommended titles in these languages
_Summary_: 


### Remove unwanted lanes from homescreen


### Autodownload content I can "check out" now when I can download it

Notes:
* automatically on wifi
* Optional on Cellular w/ setting switch in settings

### See Size of download (mbs)


### See metadata about books in catalog that's unicode compliant

### Read a book with unicode encoded characters (To research epub3 standard)

***



***

## Library (institution) Users Stories

### [Provide Recommendation from Bookish/because we don't have our own recommendation list]
* https://app.asana.com/0/12956401148094/13058132459494
#####Confirmation Criteria
* the app subscribes get feeds from Bookish API

### [Connect to my ILS/So I don't have to manage multiple Patron records]
* https://app.asana.com/0/12956401148094/13058132459500
#####Confirmation Criteria
* The app can connect to the III ILS Patron API
* The App can use Polaris API for users

### [Connect to my OPAC/So it shows content available via my catalogue]
* https://app.asana.com/0/12956401148094/13058132459507
#####Confirmation Criteria
* The App can use the BibCommons or Encore API

### Use Licensed Content/So I can provide content users want
* https://app.asana.com/0/12956401148094/13164366681665
#####Confirmation Criteria
* The App can use content hosted by Overdrive, 3M, or BiblioCommons

### Use Open Access Content/So I can access free content
* https://app.asana.com/0/12956401148094/13164366681669
#####Confirmation Criteria	
* The App can use content from HaithiTrust
* The App can use content from University Press
* The App can use content from The Guttenburg Project
* The App can use other content from other Public Domain Work repositories [see Content Sources](https://github.com/NYPL/Simplified-docs/wiki/ContentSources)

### Use Subscription Content Service Providers/Use Scribed, Oyster and new ebook services coming online
* https://app.asana.com/0/12956401148094/13164366681674
#####Confirmation Criteria
* The App can use content from Scribed (TBD)
* The App can use content from Osyter (TBD)

### Promote Recommended Catalogue Items/Increase Circulation
* https://app.asana.com/0/12956401148094/13164366681678
#####Confirmation Criteria
* The app displays only those items that are available for the needed format
* The app tracks those recommendations viewed but not borrowed or held
* The app opens to the recommendation view

### Connect to my DRM/So I can use content I licensed or purchased
* https://app.asana.com/0/12956401148094/13164366681682
#####Confirmation Criteria
* The App can use the Adobe DLLs for Content protection (depends on Readium/Adobe SDK integration)
* The App can use Sony's compiled libraries for use with URMS
* The App can use Readium LCP code libraries for for use with URMS

### Not need Adobe DRM/So I don't have to host an Adobe CMS
* https://app.asana.com/0/12956401148094/13171109685195
#####Confirmation Criteria
* The App can use Adobe's CMS service (depends on Readium/Adobe SDK integration)
* The App can use Sony's URMS
* The App can use Readium LCP

### Use Adobe DRM/So I can use my Adobe CMS investment, use my licensed content
* https://app.asana.com/0/12956401148094/13171109685199
#####Confirmation Criteria
* The App can be directed to a unique instance of Adobe (depends on Readium/Adobe SDK integration)

### Use Adobe DRM/So I can use my distributors Adobe CMS instance and use my licensed content
* https://app.asana.com/0/12956401148094/13171109685203
#####Confirmation Criteria
* The App can be directed to a unique instance of Adobe(depends on Readium/Adobe SDK integration)

### Use Sony URMS and/or LCP/So I can acquire content direct from publishers without having to purchase an Adobe SDK or CMS license
*https://app.asana.com/0/12956401148094/13171109685207
#####Confirmation Criteria
* The App can work with Sony URMS
* The App can work with LCP
* The App can identify which DRM technology is use on the content (Content Filter)

### Use my SSO solution/So I can access my systems easily
* https://app.asana.com/0/12956401148094/13171109685211
#####Confirmation Criteria
* The app provides a configurable SSO interface or DLL
* The app saves my log on credentials on the client
* The app supports secure communication SSL, crypto key exchange, static or dynamic tokens
* The app supports Kerbos or LDAP
* The app supports OATH, CAS, or Shibboleth
* The app supports OPEN ID, OAuth 2.0
* The app supports SAML (ether HTM/SSL or SOAP/Encrypted XML methods)

### Prompt Readers to check books back in/So that we can re-loan the book
* https://app.asana.com/0/12956401148094/13171109685215
#####Confirmation Criteria
* The app prompts if the book is not been opened in 3 days
* The app prompts when user views the last page
* The app prompts after the preview has been read

### Preview a book/So we don't waste a lend license
* https://app.asana.com/0/12956401148094/13171109685220
#####Confirmation Criteria
* The preview is offered along side download
* The preview is used as opposed to a licensed lend if the preview is read


### App can lend content from 3rd party subscription databases in PDF
 * https://app.asana.com/0/12956401148094/13171109685228
_Summary_: NYPL (and most libraries) subscribe to databases like Gale Group which deliver content in PDF form. Given Readium's ability to display fixed-layout epub3, the engine should be able to render the files. Given this (and that these databases are one of the library's most bountiful sources of content), we should index database PDF content and make it available to any library user through the app.

CONFIRMATION CRITERIA:
* Users can search, download, and read GaleGroup PDFs as fixed layout Epubx
* Users can search, download, and read ProQuest PDFs

Other areas:
This is dependent on both SSO being completed and the completion of the EZ-Proxy database integration.