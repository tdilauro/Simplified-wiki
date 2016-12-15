The original Library Simplified team has significant experience connecting circulation managers to III Sierra ILSes, since both the New York Public Library and the Brooklyn Public Library use Sierra as their ILS system. We've identified a number of pitfalls that can make it difficult to even know which mechanism to use.

Sierra offers two APIs that can be used to connect a circulation manager to a Sierra installation:
 - The [[Millenium]] Patron API
 - [[SIP2|SIP]]

Sierra also offers (at least) two APIs that _cannot_ be used to connect a circulation manager to a Sierra installation:
 - The [["Sierra REST API"|Sierra-REST-API]] does not allow a circulation manager to represent a patron to a third party (such as Overdrive).
 - The "PatronIO" SOAP API is used for _creating_ patron records. It is not useful for validating patron credentials.

If you have created a custom SOAP API for some other purpose, it's theoretically possible to connect a circulation manager to your ILS through that API, but you'd have to write code to talk to that custom API, and there's probably a better way.

The names can get confusing. The Millenium Patron API is a "REST API" provided by Sierra, but it is not the same as the product called "The Sierra REST API". The PatronIO SOAP API is an API for dealing with patrons, but it is not the product called the "Millenium Patron API". The Millenium Patron API is not "The III Patron Update Service" or "The My Millenium Service".

When in doubt, file a support request with Overdrive (assuming you have an Overdrive account) and ask them how Overdrive communicates with your ILS to provide ebook service.

# Sierra - Millenium Patron API

Here's an example for a common case: a library that authenticates patrons using Sierra's Millenium Patron API.

```
"authentication": {
    "providers": [
        { "module": "api.millenium_patron",
          "url": "https://my-library.iii.com:4500/PATRONAPI",
          "test_username": "12345678901234",
          "test_password": "5678",
          "authorization_identifier_blacklist": ["lost"]
        }
    ]
}
```

Here's what the keys mean:

* `module`: (REQUIRED) Indicates that authentication happens through the Millenium Patron API (as opposed to some other method).
* `url`: (REQUIRED) The root URL for your ILS's Millenium Patron API.
* `test_username` and `test_password`: (OPTIONAL) A barcode and PIN that will work against the ILS. The circulation manager uses this to test the availability of third-party APIs.
* `authorization_identifier_blacklist`: (OPTIONAL) At NYPL we noticed a lot of patrons with multiple barcodes, some of which looked like like "12345678901234LOST". This indicates that their barcode _used_ to be 12345678901234, but they lost their library card, and when they were issued a new one the librarian stuck "LOST" on the end of their old barcode instead of deleting it from the system. So we introduced `authorization_identifier_blacklist`. It helps us see when a piece of data in the barcode field is actually a note about a prior barcode, not an active barcode.