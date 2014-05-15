# Mobile Client Views

##### Home Page View

###### Navigation Elements

* Browsable list - interactions (swipe, hold or tap select)
* Search - tap for search parameter input (tap keyboard, speech?)
* Browsable "Swim Lanes" - Lists of books - interactions (swipe up down) 

###### Information Elements

* Current Login State / User
* Library
* Checked-out Books Jackets, title author (maybe due date meter?)
* Recommended Book Jacket, title author (availability?) 

##### User Profile View

###### Navigation Elements

* Back or main application menu (this can be an object or interaction)

###### Information Elements

* User Image/Gravatar (from Social Network)
* Current Login State
* User info (Names, B-Day Address, affiliated branch)
* Library and Library Card Bar Code (Scannable)
* eBooks on Holds Book Jacket, title author (availability?) 
* 3rd Party Apps linked (G+, Twitter, FB.) 



##### Registration View
This view is intended to allow new or existing users of the library to link their app to their existing accounts for Overdrive, 3M, Axis 360, Adobe, BiblioCommons/NYPL.  This view should facilitate users creating an account on those systems if required in order to transact content.  Normally, SSO would facilitate access to the service providers of content through an NYPL library card, however, that technical infrastructure does not exist today.  This view could also accommodate linkage of Social Media credentials.

###### Information Elements

* User Name
* User ID
* User pword
* Library Card (bar code)
* User Pin
* Birth Day
* Address
* Affiliated Library/Branch

###### Navigation Elements

* Service Providers - navigates to required form for credentials (assumes no look interface up from library account and that user must provide information.)
* Back or main application menu (this can be an object or interaction)

##### User Setting View
This view provides information regarding preferences of the user such as notification and alerts.  It could also potential provide self described preference for reading such as preferred genera, subjects or authors..

###### Information Elements

* Notifications options (App Badges, Text/SMS, email, none)
* Notification address (sms, email)

###### Navigation Elements


##### Reading View

###### Information Elements

* Book Title
* Book content
* Location in content (Chapter, percent read or page if applicable)

###### Navigation Elements

* Reader Settings (font size, day or night reading, spine access, brightness, font type?)
* Interaction - page turn/scroll
* Interaction - content zoom or media focus
* Application Navigation

##### Reader Application Settings

* Change font size
* Change font face (likely just "serif" or "sans-serif")
* Switch between white-on-black, black-on-white, and black-on-sepia

# Client Book States

With the following scheme, a book is always in *exactly one* of the following states:

#### (NONE) I can:

- Add Book X to my saved list. → SAVED
- Wait in line for Book X if it's not available. → WAITING FOR LOAN
- Take out a loan for Book X immediately if it's available. → NOT DOWNLOADED

#### (SAVED) When I have Book X in my saved list, I can:

- Remove Book X from my saved list. → NONE
- Wait in line for Book X if it's not available. → WAITING FOR LOAN
- Take out a loan for Book X immediately if it's available. → NOT DOWNLOADED

#### (WAITING FOR LOAN) When I'm waiting for Book X, I can:

- Elect to stop waiting for Book X. → NONE
- Successfully complete the wait for Book X and receive a loan offer. → LOAN OFFERED

#### (LOAN OFFERED) When a loan has been offered, I can:

- Accept the loan offer for Book X. → NOT DOWNLOADED
- Decline the loan offer for Book X. → NONE
- Do nothing, and let the loan offer for Book X expire. → LOAN OFFER EXPIRED

#### (LOAN OFFER EXPIRED) When the loan offer for Book X has expired, I can:

- Dismiss the notification that the loan offer has expired. → NONE
- Dismiss the notification that the loan offer, then go to the library to get it again. → NONE

#### (NOT DOWNLOADED) When I have been issued a loan for Book X, I can:

- Place Book X is in the download queue if other books are downloading. → DOWNLOAD QUEUED
- Begin downloading Book X if there is no download queue. → DOWNLOADING
- Return Book X. → NONE
- Allow the loan for Book X to expire. → LOAN EXPIRED

#### (LOAN EXPIRED) When the loan for Book X has expired, I can:

- Dismiss the notification that the loan has expired. → NONE
- Dismiss the notification that the loan has, then go to the library to get it again. → NONE

#### (DOWNLOAD QUEUED) While the download for Book X is queued, I can:

- Automatically begin downloading Book X once other books are finished. → DOWNLOADING
- Cancel the download for Book X. → NOT DOWNLOADED
- Fail the download due to loss of connectivity. → DOWNLOAD FAILED

#### (DOWNLOADING) When I'm downloading Book X, I can:

- Wait for the download for Book X to complete successfully. → DOWNLOAD COMPLETED
- Cancel the download for Book X. → NOT DOWNLOADED
- Fail the download due to loss of connectivity. → DOWNLOAD FAILED

#### (DOWNLOAD FAILED) When the download for Book X has failed, I can:

- Place Book X is in the download queue if other books are downloading. → DOWNLOAD QUEUED
- Begin downloading Book X if there is no download queue. → DOWNLOADING
- Return Book X. → NONE
- Allow the loan for Book X to expire. → LOAN EXPIRED

#### (DOWNLOAD COMPLETED) Once the download for Book X has completed successfully, I can:

- Open Book X. → BOOK OPENED
- Return Book X. → NONE
- Allow the loan for Book X to expire. → LOAN EXPIRED

#### (OPENED) Once I have opened Book X, I can:

- Return Book X. → NONE
- Allow the loan for Book X to expire. → LOAN EXPIRED