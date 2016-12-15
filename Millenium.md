The Millenium Patron API is a simple HTTP-based API that returns HTML documents. It is the recommended technique for getting a Library Simplified circulation manager to communicate with the Sierra ILS.

Millenium Patron API URLs tend to look like this:

* `https://ils.mylibrary.org:54620/PATRONAPI/`
* `https://mylibrary.iii.com:54620/PATRONAPI/`
* `http://ils.mylibrary.org:4500/PATRONAPI/`
* `http://mylibrary.iii.com:4500/PATRONAPI/`

We'll want to use the HTTPS URLs (which tend to include port 54620), since the circulation manager will probably be calling out to your ILS across the public Internet. If you don't have an HTTPS version of the Millenium Patron API, file a support request with III and ask for one.

The Library Simplified circulation manager uses two endpoints of the Millenium Patron API:

* `pintest`: Verify that a barcode/PIN combination is valid. Example URL: `https://ils.mylibrary.org:54620/PATRONAPI/{barcode}/{pin}/pintest`

* `dump`: Return an HTML representation of a patron's ILS record. Example URL: `https://ils.mylibrary.org:54620/PATRONAPI/{barcode}/dump`

# Sample configuration

Here's how you might set up your circulation manager to communicate with your Sierra ILS via the Millenium Patron API:

```
"authentication": {
    "providers": [
        { "module": "api.millenium_patron",
          "url": "https://my-library.iii.com:54620/PATRONAPI",
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

Because you don't want random people to be able to check patron PINs and look at patron records, access to the Millenium Patron API should be restricted only to the specific IP addresses that need it. (This is a good thing to check, actually. If your library offers public wifi, get on the wifi and try to access the Millenium Patron API. You should not be able to connect!)

If your Library Simplified circulation manager is hosted outside your library's internal network, it should also be blocked. Once you have an IP address for your circulation manager, your network administrator will need to whitelist that IP address.

# SSL certificate chain

We've noticed that the hosts of the Millenium Patron API tends to have problems with their SSL certificate chains. The Library Simplified circulation manager will refuse to connect to a server unless it can connect that server's SSL certificate with a certificate it already trusts. For many III hosts, a key piece of evidence is missing, so the circulation manager can't make this connection.

You can test for this problem by trying to access the Millenium Patron API using the command-line `curl` command:

```
$ curl https://ils.mylibrary.org:54620/PATRONAPI/
```

If you get this error, you need to have III fix the SSL certificate chain:

```
curl: (60) Peer's Certificate issuer is not recognized.
More details here: https://curl.haxx.se/docs/sslcerts.html
```

To fix this problem, file a support ticket with III and mention NYPL's III ticket #370916. 

While you're waiting for III to fix the certificate, you can deactivate certificate checking for Millenium Patron by setting the `verify_certificate` configuration option to `false`:

```
"authentication": {
    "providers": [
        { "module": "api.millenium_patron",
          "url": "https://my-library.iii.com:54620/PATRONAPI",
          "verify_certificate": false
        }
    ]
}
```

This is a security risk and you should remove the `verify_certificate` line before putting your server into production.