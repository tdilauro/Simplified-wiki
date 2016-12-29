# OneClick Integration
This document is to help you connect your library's OneClick collection to your Library Simplified circulation manager.


# Table of Contents
* [Getting Access](#getting-access)
* [API Basics](#api-basics)
* [Patron Account Management](#patron-account-management)
* [Start Your Catalog](#start-your-catalog)
* [Maintain Your Catalog](#maintain-your-catalog)
* [Book Availability](#book-availability)
* [Search The Catalog](#search-the-catalog)
* [Checkouts](#checkouts)
* [Holds](#holds)
* [Wishlists](#wishlists)
* [Bookmarks](#bookmarks)
* [Fulfillment](#fulfillment)
* [DRM](#drm)
* [Testing](#testing)


## Getting Access:
OneClick provides trusted partner tokens.  You'll need one to talk to their api.  To obtain your tokens, you'll need to contact OneClick via email or Slack.

OneClick API can communicate through HTTPS.


## API Basics:
OneClick provides [API Documentation](http://developer.oneclickdigital.us/documents/getting-started). 

Base API URLs are:
- api.oneclickdigital.us for QA  
- api.oneclickdigital.com for Prod  

The API attempts to be as RESTful as possible.  Most request parameters are passed in the URL, and the http method the request is sent by helps differentiate between related request types.  For example, let's say you have a library whose OneClick id is 123.  In that library's OneClick collection, you have a book with the ISBN number 456.  And you have a patron with OneClick id 789.  Then, to ask to check out this book to this patron, you would sent a POST request to the URL [api_url]/123/patrons/789/checkouts/456.  And to ask to return the book, you would send a request to the same URL, but through a DELETE http method.

Most OneClick responses are in the JSON format.  You'll either get a data structure containing the information you asked for, or a dictionary with an error message.  Here, OneClick puts HTTP response codes to intensive use.  Bad requests, data conflicts, books not found are all indicated through HTTP 500, 404, 409, etc. codes.  The error message then provides a more detailed explanation of the problem.  For some types of requests, you can get either an empty-bodied response or a non-JSON single word or number response.  This is relatively rare.

Many queries can specify the verbosity of the JSON response they want to get.  This is accomplished by sending an "Accept-Media: [verbosity-level]" header in the request.  Verbosity level options are: {0:'basic', 1:'compact', 2:'complete', 3:'extended', 4:'hypermedia'}.

You will also be sending your trusted token in as a "Authorization: Basic [your-token-here]" header with each api query.


## Patron Account Management:
We store patron account data in our ILSes, and pass it through to OneCLick.  OneClick stores their own version of patron records, with the patron data we supply, plus a OneClick internal patron id number.  Thus, our ILS is the One True Source to keep track of.  You will use the data from the ILS to ask OneClick for its internal patron id.  You'll then use that internal patron id in transactions on behalf of the patron.  The OneClick internal patron id will not change over the course of the account, barring major migrations.  But it's still good policy to re-get it with the first step.

For patrons who already have OneClick accounts, the account that Library Simplified creates is not merged with the pre-existing account.  The Library Simplified OneClick account is done seamlessly to the patron.  If the patron notices a discrepancy and calls tech support, the accounts can be manually merged.

To read a patron's OneClick record, send a GET call to [api_url]/[library_id]/patrons/[patron_id] .  To make a patron, the code sends a POST request to that url.


## Start Your Catalog:
You've made a database:   
```sql
CREATE DATABASE my_circulation_db_name;
GRANT all privileges on database my_circulation_db_name to my_db_user;
```
   
You've filled in the Postgres and the OneClick integrations blocks in your config.json:
```json
[...]
    "integrations" : {
        [...]
        "Postgres" : {
            [...]
            "production_url" : "postgres://my_db_user:xyz@localhost:port-number/my_circulation_db_name"            
        },
        [...]
        "OneClick" : {
            "username" : "your-oneclick-given-username",
            "password" : "your-oneclick-given-password",
            "library_id" : "your-oneclick-given-4-digit-id",
            "remote_stage" : "qa", 
            "url" : "https://api.oneclickdigital.us/", 
            "basic_token" : "your-trusted-partner-token", 
            "ebook_loan_length" : "two-digit-number-in-days", 
            "eaudio_loan_length" : "two-digit-number-in-days"
        },
    }
[...]
```

To know what titles to show your customers, you'll need to populate your database with your library's OneClick catalog.  You can get this by running the import script in your virtual environment.

If you don't already have your virtual environment running, create it, and run then start it up with:
```
source circulation_env/bin/activate
```

Anyways, the import script:
```bash
(circulation_env)$ python bin/oneclick_library_import
```

The script sends a GET request to [api_url]/[library_id]/media/all .  **This request is quite taxing on OneClick servers, so use sparingly and don't DOS them :).**

OneClick currently has collections of ebooks, eaudio, and emagazines.  Library Simplified currently handles ebooks, with eaudio in the pipeline.  The magazine collections are handled through a separate API on the OneClick side, and are to become more integrated in the future.  They are further removed in the Library Simplified's pipeline.


###Availability
You've got the catalog metadata imported, but you don't yet have the info on which books are currently available for download.  Without it, your patrons can't borrow.  Request this information from OneClick with:
```
python bin/oneclick_monitor_availability 
```

Note:  You'll probably want to run this script on a regular basis, to keep updating book availability information.

## Maintain Your Catalog:
Once the catalog is loaded into the database, you'll need a way to check for and import any changes.  Do this with the 
```
(circulation_env)$ python bin/oneclick_library_delta
``` 
script.  The script will call the [api_url]/[library_id]/media/delta?begin=YYYY-MM-DD&end=YYYY-MM-DD endpoint, described in the API [here](http://developer.oneclickdigital.us/endpoints/titles#get-calculated-deltas).  The delta calls date ranges are limited to last couple of months.


## Book Availability:
There is no way to tell how many licensed copies of a book are available for lending.  All we can know, is the binary answer to whether the book was available when the last catalog availability check was run.  If the book is marked "available", then the patron may try to check it out.  If the book's licenses have gotten used up in the time between our availability check and the checkout request, an appropriate error message will be displayed, the patron will have the opportunity to put the book on hold, and the availability flag in our database will be updated.

There are some, very very rare libraries that choose to display items they do not own in their OneClick collections.  This option is turned on by OneClick on request, when a library account is set up.  For such catalogs, the non-owned items can appear in search (the "interest" field in their metadata will reflect the outside catalog ownership), and can be put on wishlist, but not checked out.  If an item is marked "available" in its metadata, it is in fact also a guarantee that the library owns it and it's clear for checkout.


## Search The Catalog:
The API call is described [here](http://developer.oneclickdigital.us/endpoints/search-partner).  Your search capabilities are not completely flexible -- there is a set list of facet and filter values to search by.  For more flexibility, you'll be using the Library Simplified search functionality, rather than the direct call to OneClick API.

Book categories that are used as search facet values are determined by OneClick, and are generated automatically on ingestion into OneClick, with BISAC codes mapped into 34 OneClick genres. The original codes are sometimes but not always kept on in the MARC records.

Audience determination comes down from publisher, and is in 4 groups. There is an age rating for children's, but not as part of audience rating.


## Checkouts:
Resources are identified by library id, patron id, and item id.  The library and patron ids are OneClick-internal.  The item id is usually an ISBN.  See [here](http://developer.oneclickdigital.us/endpoints/title-checkouts-admin) for the list of available functionality.  

The checkout process on the OneClick side is:
- Look at library id. Is patron authorized for that library?
- Look at patron, are they maxed out? Max checkouts are set in their admin interface by the library itself.
- Assume the patron is already authenticated by Library Simplified, the trusted partner.
- Resource id is always isbn.
- Renewing restrictions – are a setting in each library’s admin interface.

If the book is not available, or if the patron is not allowed to check out (for example, if they've reached their book quota), you'll get an HTTP error status and a response text with a more detailed error message.  

An HTTP POST call performs a checkout.  A DELETE call returns a book.  And a GET call returns the patron's current checkouts.

The book is checked out for X number of days, where X is determined by your library's agreement with OneClick.  You can shorten the amount of time a book is checked out for by passing the optional "days" parameter.  You cannot extend the checkout time range with the "days" parameter past the library's default X.  You will be able to extend the loan by sending a renew call.

Audiobooks are multi-use unlimited subscriptions.
Eboooks are single-use 1 copy, but some have catalog-level expirations, which will see in deltas.


## Holds:
Work like checkouts, but with books in the catalog that are not currently available.  If you attempt to put a hold on a book that is currently available, your hold request will succeed, and then a bit later, a OneClick daemon will silently convert the hold record to a checkout record.  Holds don't expire, they just convert to checkouts when the items become available. 


## Wishlists:
These exist to help libraries know which books their patron would like them to acquire.  Library Simplified does not currently implement this request functionality.


## Bookmarks:
Currently the functionality is limited to audiobook playback positions.
Assume an audio book is available in one format/encoding quality.


## Fulfillment:
First, you make a call to get all checkouts for the patron.  The response has checkout metadata, including fulfillment urls.  These urls must be called to obtain time-sensitive download/stream links to actual ebook or eaudio files.  The download urls expire in about 15 minutes.  For some eaudio books, there can also be a "downloadUrl" property on the checkout, which allows you to download a zip file containing all audio files for the book.

EBooks are all protected with Adobe DRM, so the download link gets an ASCM (Adobe Content Server Message) file.  The acsm file is essentially a "token" file that contains all the necessary information for Adobe Digital Editions to open an ebook.  

EAudio books are either un-drm-ed (those are the ones you can download a zip for), or protected by OneClick's proprietary drm.  Audiobooks are always in mp3 32 bit mono format.


## DRM:
There are two fields to look for in the item metadata: hasDigitalRights and hasdrm.  hasDigitalRights is for ebooks, hasdrm is for audiobooks (???).  All ebooks use AdobeDRM (acsm files).  Audiobooks come in two flavors: non-encrypted (fulfillment allows download of zip file) and with OneClick proprietary DRM (fulfillment sends time-sensitive download link).


## Testing:
To make a test patron, you can take several routes.  You can send a create-patron api call.  Or you can go to One Click's website for your library or consortium.  Here's the [QA interface link] (http://iconnct.oneclickdigital.com/#register) for Connecticut.

