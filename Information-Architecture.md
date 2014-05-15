### Mobile Client Views

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

### Client Book States

![chart with possible book states. states with the same color are mutually exclusive.](https://www.dropbox.com/s/8fi1tpxwsbx0ec6/book%20states.png)

Books are always in one of the following states, each of which should have an associated visual indicator:

* **Permanent:** These are books which the user has downloaded that can be retained indefinitely. Such books will likely primarily be public domain titles, e.g. those obtained via Project Gutenberg.
* **On loan:** These are books for which the user has taken successfully taken out a loan. An indicator may reveal the number of days left on the loan without having to view the book's details.
* **Awaiting loan decision**: These are books for which the user has requested a loan at a time when a copy was not available but that has since become available. The user has a set number of days to make a decision as to if she will accept or decline the loan.
* **Loan expired**: These are books which have expired due to the user not opting to return the book before the end of the loan term. These books remain for two reasons: to avoid the potential confusion that would result from a book suddenly disappearing, and to provide the user with a convenient way to request another loan.
* **Loan decision expired:** Similar to the previous state, this state indicates that a loan was offered but neither accepted nor declined.

In addition to the above states, a given book may have one of the additional states:

* **Downloading:** These are books for which a download is currently in progress.
* **Download completed:** These are books that have been successfully downloaded by not yet interacted with in any way. The purpose of this state is to highlight that the book is newly available to the user and will be cleared upon opening the book (at a minimum).
* **Download failed:** These are books for which a previously initiated download has since failed and has not yet been retried.
* **Download queued:** These are books for which the user has requested downloading that have not yet begun downloading due to a limitation on simultaneous downloads.
* **Download paused:** These are books for which the user has temporarily paused downloading with the option of resuming later.
* **Book opened:** These are books in which the user has navigated away from the first page. The intent of this state is to make it clear to the user that opening the book will bring them directly to where they left off rather than the default location.