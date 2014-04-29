## Requirements

* Checking out books from Overdrive or (possibly) Axis 360 requires
  that we know the user's barcode and PIN. If there is no way to look
  up a patron's PIN, that information must come from the patron. This
  means a system where users are expected to know their barcode and
  PIN, and provide it to us as their primary credential.
* We need to be able to create a new patron account, with a barcode.

To integrate with nypl.org accounts we also need the following:

* Validate nypl.org username and password
* Look up barcode and PIN for nypl.org username

If we can have these two features we can use nypl.org username and
password as the primary credential, and quietly manage barcode and PIN
behind the scenes.

## 3M

Signup: Signup happens out of band. (Inside the 3M reader app? What
information is requested? Probably barcode+PIN.)

Authentication: 3M's API authenticates the library. The library may
take actions on behalf of any patron. For instance, the library may
make a Checkout call for a patron by including the patron's barcode in
the "PatronId" tag.

Behind the scenes, 3M must be storing barcode/PIN on server side, and
validating barcode/PIN with NYPL's system before letting a request go
through. How are they doing this?

EXPERIMENT: Can we place a hold on a book on behalf of a PatronId who
has never used the 3M Reader?

## Overdrive

Signup: Is there any explicit signup? When I go to ebooks.nypl.org I'm invited to sign in with barcode+PIN.

Using the [Checkouts
API](https://developer.overdrive.com/apis/checkouts) or the [Patron
Information
API)(https://developer.overdrive.com/apis/patron-information) on
behalf of a patron requires authenticating as that patron. We can get
an OAuth token for an individual patron using the [Patron
authentication API.](https://developer.overdrive.com/apis/patron-auth)
This requires providing the patron's barcode and PIN. The OAuth token
expires after one hour.

Behind the scenes, Overdrive is proxying barcode+PIN to NYPL's
system to verify correctness.

EXPERIMENT: Can we get a patron OAuth token for a patron who has never
used Overdrive, using only barcode+PIN?

## Axis 360

Axis 360 seems to support both an Overdrive-type system and a 3M-type
system, depending on a setting called the "patron validation
flag". When the flag is set, making API calls requires going through a
system called "Patron Auth" which is very similar to Overdrive's
"Patron Authentication API". This system takes barcode and PIN as
input. But when the flag is not set, it's like 3M. The library can act
on behalf of any patron, without authenticating.

There is an API call called "DRM Create" which creates a new DRM
account for a patron. I'm not sure what this does, but it looks like
you need to set up the system ahead of time with whatever DRM
providers you're using (e.g. Blio or Acoustik).

## Bibliocommons

Bibliocommons user accounts are nypl.org user accounts. I don't know if this is because BC is integrated with our database or because BC manages our user accounts. 

If you know the nypl.org username you can look up the user record using the BC API (the /users search). However this includes no useful information AFAICT. The BC user ID is not the same as the NYPL barcode.

## Innovative Patron API

This is an API to our internal patron database. [The
documentation.](http://vendordocs.iii.com/) (user/pass:
nypl_s/chapter) It's not clear whether we actually have access to this
API. If so, I don't know the hostname or port of the endpoint we can
use.

### Patron creation

There's [a patron creation call](http://techdocs.iii.com/patronws_api_operations.shtml#createPatron) but I have no idea what the HTTP method looks like or whether we have this enabled. 

### PIN Validation

We can validate a patron's PIN by sending a GET request to
[https://{site}/PATRONAPI/{barcode}/{PING}/pintest](http://partner.iii.com:4500/PATRONAPI/20879875432970/1234/pintest).

If the PIN is valid, an HTML document containing the following data:

   RETCOD=0

If invalid, the data looks like this (it's separated by BR tags): 

   RETCOD=1
   ERRNUM=4
   ERRMSG=Invalid patron PIN

### Patron data

We can get detailed information about a patron's library account by
sending a GET request to
[http://{site}/PATRONAPI/{barcode}/](http://partner.iii.com:4500/PATRONAPI/20879875432970/dump).

Sample URL:

"20879875432970" here is the 14-digit barcode number.

The response is an awkwardly-formatted HTML document:

    REC INFO[p!]=p
    EXP DATE[p43]=  -  -  
    PCODE1[p44]= 
    PCODE2[p45]= 
    PCODE3[p46]=0
    P TYPE[p47]=2
    TOT CHKOUT[p48]=259
    TOT RENWAL[p49]=12
    CUR CHKOUT[p50]=14
    BIRTH DATE[p51]=  -  -19  
    HOME LIBR[p53]=main 
    PMESSAGE[p54]= 
    MBLOCK[p56]=-
    REC TYPE[p80]=p
    RECORD #[p81]=1002752
    REC LENG[p82]=1273
    CREATED[p83]=06-10-08
    UPDATED[p84]=04-29-14
    REVISIONS[p85]=2404
    AGENCY[p86]=1
    CL RTRND[p95]=1
    MONEY OWED[p96]=$0.00
    CUR ITEMA[p102]=2
    CUR ITEMB[p103]=0
    ILL REQUES[p122]=2
    CUR ITEMC[p124]=0
    CUR ITEMD[p125]=0
    CIRCACTIVE[p163]=04-29-14
    LANG PREF[p263]=   
    NOTICE PREF[p268]=z
    WAITLIST[p297]=0
    HOLD[p8]=P#=1002752, I#=1011894, P=08-09-10, NNB=08-09-10 (0 days), RLA=0, NNA=01-17-38, ST=0, TP=b, PU=main 
    HOLD[p8]=P#=1002752, I#=1137241, P=05-03-12, NNB=05-03-12 (0 days), RLA=0, NNA=01-17-38, ST=105, TP=i, PU=main 
    HOLD[p8]=P#=1002752, I#=1040403, P=06-08-12, NNB=06-08-12 (0 days), RLA=0, NNA=01-17-38, ST=105, TP=i, PU=     
    HOLD[p8]=P#=1002752, I#=1000033, P=09-10-12, NNB=09-10-12 (0 days), RLA=0, NNA=01-17-38, ST=105, TP=i, PU=     
    MESSAGE[pm]=PONY!
    PATRN NAME[pn]=Leckbee, Eric
    ADDRESS[pa]=$$$
    TELEPHONE[pt]=510-655-5200
    NOTE[px]=SATAN!
    NOTE[px]=PIN is 1234
    NOTE[px]=Fri Jul 29 2011: Claimed returned .i1259509 on Fri Jul 29 2011
    MULTI[p1]=
    P BARCODE[pb]=20879875432970
    PIN[p=]=sZEtR.yTgJVaM
    EMAIL ADDR[pz]=ericleckbee@yahoo.com
    COURSES[ps]=MATH123
    LINK REC[p^]=i

Some useful fields:

 * BLK UNTIL: The date until which the patron is blocked from using library services.
 * EXP DATE: The expiration date of the patron's borrowing privileges.
 * LANG PREF: Preferred language
 * P BARCODE: The patron's barcode
 * PATRN NAME: The patron's name
 * PIN: Optional field containing the patron's PIN in encrypted
   form. Since there are only 9999 possible PINs, cracking this is
   pretty easy, but a) we may not have this field, and b) that's
   a dumb way to get the PIN.

 Note that in the example document, MESSAGE[px] also includes the
 plain text "PIN is 1234". Again, no guarantee our system does that.

### III  - Patron APIs

*This is old writing and has not been brought up to date.*

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

#### Existing card provisioning APIs

The Brooklyn Public Library created a program called [My Library NYC](http://mylibrarynyc.org/about), in which you can sign up online
for a unified library card that works with all NYC libraries. What API
are they using to create a NYPL credential?
