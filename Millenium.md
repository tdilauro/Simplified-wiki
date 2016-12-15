The Millenium Patron API is a simple HTTP-based API that returns HTML documents. 

Millenium Patron API URLs tend to look like this:

* `https://ils.mylibrary.org:54620/PATRONAPI/`
* `https://mylibrary.iii.com:54620/PATRONAPI/`
* `http://ils.mylibrary.org:4500/PATRONAPI/`
* `http://mylibrary.iii.com:4500/PATRONAPI/`

We'll want to use the HTTPS URLs (which tend to include port 54620), since the circulation manager will probably be calling out to your ILS across the public Internet.

The Library Simplified circulation manager uses two endpoints of the Millenium Patron API:

* `pintest`: Verify that a barcode/PIN combination is valid. Example URL: `https://ils.mylibrary.org:54620/PATRONAPI/{barcode}/{pin}/pintest`

* `dump`: Return an HTML representation of a patron's ILS record. Example URL: `https://ils.mylibrary.org:54620/PATRONAPI/{barcode}/dump`

# Sample configuration

Here's how you might set up your circulation manager to communicate with your Sierra ILS via the Millenium Patron API:

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

# IP Whitelisting

# SSL certificate chain