***

### [User can sign into app using existing NYPL Credentials]

* (https://app.asana.com/0/12956401148094/12956401148102)[James]

_SUMMARY DESCRIPTION:_ An existing NYPL user with a valid, active library card can use their existing credentials [BARCODE + PIN] to authenticate to the app, and perform any functions which require an account. 
***

### [Get a Library Card/Register I can Borrow eBooks]	
_SUMARY DESCRIPTION_: Settings View of app allows a user app for a library card by providing the user the ability to provide (Name, DOB, Address, User Name, Password, PIN, email, Phone Number)
#####Confirmation Criteria
* The user can log into other digital properties with their newly established credentials
* Information from the form is stored in the appropriate Identity Management database (SSO)
* An acknowledgment email or SMS is sent to the user after submitting the form (System TBD).
* The user's account is created in the ILS
* The user's account is created in our DRM instance
* The user's account is created in BiblioCommons
* The user's account is created in our eBook Distributor (Overdrive, 3M, Axis 360)
***

### [Connect my library account to my App/Don't have enter information again]	
_SUMARY DESCRIPTION_: This functionality can range from scanning the library barcode to simply providing the necessary credentials and confirming them with the requisite ILS or Identity Management System
#####Confirmation Criteria
* The App can use III ILS Patron API
* The App can use Polaris ILS API
* The app can use SSO (CAS, OAUTH, Shiboleth, other if required - TBD)

### [Search available titles/So I can borrow the title for free]
_SUMARY DESCRIPTION_: This is a basic search function of the eBook Collection.  However, it may eventually link to a broader search of the catalogue regardless of format.
* Can enter a titles and get like title results that are available for download in my needed format
* I can search for titles by multiple facets (title, subject, author, genre) and get like title results that are available for download in my needed format
***

### [Get recommendations from Library/So I don't have to search for items]
_SUMARY DESCRIPTION_:	This is the core discovery mechanism for the eBook Collection.  However, it may eventually link to a broader set or combined set of recommendation signals to items in the catalogue regardless of format.
* See available titles recommended by Staff (Physical or Digital)
*  See available ebook (Digital) titles that are recommended, down loadable, and of the right digital format for my device

### [Connect to my 3rd party Reading list/So I can record my reads]
_SUMARY DESCRIPTION_:Users who are part of online book clubs or actively manage their reading can link the app to those existing venues for recording their reading history or story in their books
#####Confirmation Criteria
* The App can transact on my Google Books Reading List
* The App can transact on my Good Reads books shelf API
* 3rd Party Book List Integration
***

### [Create Wish List/So I can get notifications when the book is available]
_SUMARY DESCRIPTION_:This allows users to create lists of book they may want to borrow from the library
#####Confirmation Criteria
* The App can use my Google Books Reading List to identify wished for books and whether they become a part of the NYPL collection
* The App can use my Good Reads books shelf API to wished for books and whether they become a part of the NYPL collection

### [Be recommended only titles that are available/So I don't have to browse through titles not available to read now]
_SUMARY DESCRIPTION_:This feature or setting allows discovery of books that can be accessed immediately by the reader with out having to wait in hold queues.
#####Confirmation Criteria
* I get only Titles in our Catalogue/Collection (Not sold by our platform vendor but unlicensed by NYPL)
* I get Titles in the format my device and app supports (If content is only in Kindle, and I am non iPhone don't bother)
* I get only titles that have an available copy 

### [Borrow a physical copy if the eBook isn't available/So I don't have to wait to read the book]
_SUMARY DESCRIPTION_:Based on the popularity of the searched for title, I may be willing to get an alternate digital version (Audio) or  physical copy.  I would like to see the alternate versions held in the broader collection
#####Confirmation Criteria
* I can place a hold on the physical format if the digital format is not available
* The hold reservation is in the ILS/or digital CMS
* The physical title copy is available for pick-up at branch
* The alternate formats are displayed in the apps title detail page (Assume all searches are for eBooks)
***

### [Find and Download the App/So I can try the app]
_SUMARY DESCRIPTION_:This an ability of the app to be rapidly acquired or distributed by users
#####Confirmation Criteria
* The App is In the Apple App Store
* The App is in Google Play (if applicable)
* The App is on the NYPL.org
* The App download location can be shared by a user (email, Social Network)
***

### [Use the App on Smart Phone]
#####Confirmation Criteria
* The App can be installed on an iPhone
* The App can be installed on an Android
* The App can be installed on Windows Phone
***

### [Use the App on my Tablet]
* The App appropriately uses the screen real-estate
* The App adjust the views as necessary to not require pixilation
* The Can render content without formatting issues
* The App can be installed on iOS
* The App can be Installed on Android (TBD)
* The App can be Installed on Windows Tablet (TBD)

### [Create and account using my email credentials/Don't have enter information again]
_SUMMARY DESCRIPTION_:This allows users to sign up for the app using their establish credentials for email or social network like Facebook, Google or Yahoo.  This also allows app to capture data needed for library card registration.
#####Confirmation Criteria
* The App can use Oauth2.0 or OpenID for GMAIL
* The App can use SOAP for MSN (?)
***

### [Create an account using my device credentials/Don't have enter information again]
_SUMMARY DESCRIPTION_: This allows the user to establish their credentials using the already established credentials on the platform the are using.
#####Confirmation Criteria
* The App can us my iCoud ID (iOS)
* The App can use my SIM ID and or Android Device ID (if applicable)
***

### Homescreen has scrollable Lanes of available material [James]
_SUMMARY DESCRIPTION_: Home View of app is "store" with covers of books available to borrow (or download) now, scrollable from side-to-side sorted by categories.

***

### 