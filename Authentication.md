### III  - Patron APIs

A patron may have two different credentials:

* A nypl.org username and password
* A library card barcode and PIN

To use an analogy from Jason Varghese, the username/password is your account on a bank website, and the barcode/PIN is your ATM card. We are developing an "ATM", so our primary authentication mechanism will be barcode/PIN. 

#### Barcode authentication

_The SSO project will provide an API for sending barcode/PIN and getting a yes/no response._ This will let us do initial authentication.

Our ebook providers accept barcode/PIN and turn around to authenticate it with ILS, so our server implementation needs to have barcode/PIN information to send to the ebook providers. There are a 
number of ways this can work:

1. The client can store this information from initial authentication and provide it to the server with every request that needs authentication. (We will verify it with SSO every time.)
2. The server can keep this information from initial authentication. We will verify it only once.
3. The server can use the initial authentication to get a token from SSO. We will use that token to request barcode/PIN from SSO whenever we need to pass it on to the ebook providers. This is the same as #1, but it means we don't store the barcode/PIN on our server.
4. We can get the ebook providers to stop authenticating against ILS and start authenticating against SSO. We will then pass some information to the providers other than barcode/PIN.

In the first two cases, if the user changes their PIN, the next authenticated call will fail and we will ask the client to authenticate again.

#### Username authentication

Users who have both a nypl.org account and a library card may prefer to log in using username/password rather than barcode/PIN. _The SSO project will provide an API for sending username/password and getting barcode/PIN in response._ If we switch over to using a token instead of passing around barcode/PIN, we will get one of those tokens instead of getting barcode/PIN.

#### Current authentication APIs

The [Innovative Patron
API](http://vendordocs.iii.com/patron/patronapi.shtml) (user:pass
nypl_s:chapter) can validate a patron based on identifier and PIN.

The [Innovative Millenium API](http://techdocs.iii.com/patronws_patron_data.shtml) (user:pass nypl_s:chapter) probably also does
validation, but I'm not sure how.

#### New library card

This is the tricky one. Here's one current procedure for getting an NYPL library card, as tested by me:

1. Register an account at nypl.org. Set up a temporary card ID that will expire at the end of the day, and set a PIN. 
2. Go to a branch library that same day, show your temporary ID and your drivers' license, and get a physical library card. 

Some notes on the existing process:

* The physical card has a different ID from the temporary card: they have a stack of cards at the counter and they pull one off the stack.
* The PIN is not needed to exchange a temporary card for a permanent card.
* The temporary card ID is destroyed when the physical card is activated.
* The PIN for the permanent card is the same as the PIN for the temporary card.
* Unfortunately I didn't test whether I could check out ebooks with the temporary card.

_Here are the use cases we want to support._

1. Anyone who boots up the app from a NYC GeoIP should be able to get a permanent virtual library card by providing their date of birth and address. They can go to a branch library at any time if they want to exchange their virtual library card for a physical library card.
2. Anyone who boots up the app from outside NYC should be able to get a temporary (30-day) virtual library card by providing their date of birth and address. Within that 30-day period they can go to a branch library to exchange their temporary virtual card for a "permanent" physical card.
3. Anyone who boots up the app from outside NYC should be able to get a temporary (30-day) virtual library card by providing their date of birth and address *within NYC*. A "permanent" physical card will be mailed to their address, along with instructions for activating it.

Notes:

* Obviously there are lots of edge cases here, but I believe this captures what we want: to serve people who live in New York or spend a lot of time in New York. Making it difficult to game the system, but not at the expense of making things really easy for New Yorkers.
* I put "permanent" in scare quotes because I believe physical cards are only good for three years, and have to be renewed. If this is true, we'll need additional use cases for renewing virtual cards.
* #1 and #2 are almost the same. The difference is when you verify that you spend time in NYC. If you're in NYC right now, your virtual card is permanent until you exchange it for a physical card. If you're not in NYC right now, you have 30 days to get to a branch library to claim your permanent, physical card.
* #3 is similar to how Google verifies that you own a business you claim to own.
* Hopefully we can do #1 and #2 with a minimum of change to existing processes. The major change: _we need to provision virtual card numbers that are good for 30 days, or permanent, instead of the current cards that expire at the end of the day. The virtual card numbers need to have permission high enough to check out ebooks._
* #3 will require a completely new process, and IMO it can wait.
* Physical card numbers are pre-provisioned (that's the stack of cards on the counter). I don't know if the same is true for virtual card numbers. We may need to have a block of card numbers provisioned for us

_Jason proposed that SSO offer an API for provisioning barcode and PIN--either associated with an existing username/password, or associated with a brand new account._ This way we don't have to make API calls directly to ILS. One less system talking directly to ILS is one less system that has to be notified when the ILS system changes.

#### Mirroring changes to ebook providers

It doesn't sound like 3M or OverDrive need to know about our accounts--we give them barcode/PIN and they verify with ILS. Hopefully there is nothing to do here. Other ebook providers might do things differently.

#### Existing card provisioning APIs

The Brooklyn Public Library created a program called [My Library NYC](http://mylibrarynyc.org/about), in which you can sign up online
for a unified library card that works with all NYC libraries. What API
are they using to create a NYPL credential?

The [Innovative Millenium API](http://techdocs.iii.com/patronws_patron_data.shtml) (user:pass nypl_s:chapter) includes a "Patron Update Web Service" which can [create a new patron](http://techdocs.iii.com/patronws_api_operations.shtml#createPatron).
