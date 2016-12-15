

* In addition to SIP, Sierra offers a number of APIs:
 - The [[Millenium]] Patron API
 - The [[Sierra-REST-API]] REST API
 - The "PatronIO" SOAP API is used for _creating_ patron records. It is not useful for validating patron credentials.

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