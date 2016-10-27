Each library has some way of identifying and authenticating patrons. Authentication serves two purposes: to distinguish between patrons and non-patrons, and to identify resources associated with a particular patron.

Libraries most commonly authenticate patrons against an Integrated Library System (ILS). The Library Simplified circulation manager integrates with a few ILS systems through custom APIs, and with many more through the SIP2 protocol.

A library may support multiple authentication techniques. For example, the Open Ebooks collection authenticates patrons through both First Book and Clever. Currently, a circulation manager can be configured to support one HTTP Basic Auth mechanism and any number of OAuth mechanisms.

To connect your circulation manager to one or more authentication sources, define a `policies["authentication"]` section in your configuration file.

* `providers`: A list of authentication providers. At least one provider MUST be present.
* `bearer_token_signing_secret`: A secret used to sign OAuth bearer tokens before sending them to clients. If you define an OAuth provider, this MUST be present. If you define more than one OAuth provider, they will all sign keys using the same secret.
* `register_url`: If you provide this, unauthenticated patrons will be given the opportunity to sign up for an account rather than enter credentials. A patron who chooses to sign up for an account will be sent to the given URL. TODO: This is not implemented yet, and there are several undecided issues surrounding it.

Each authentication source needs its own configuration. They are covered individually below.

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

* `module`: (REQUIRED) Indicates that authentication happens through the Millenium Patron API (as opposed to SIP).
* `url`: (REQUIRED) The root URL for your ILS's Millenium Patron API.
* `test_username` and `test_password`: (OPTIONAL) A barcode and PIN that will work against the ILS. The circulation manager uses this to test the availability of third-party APIs.
* `authorization_identifier_blacklist`: (OPTIONAL) At NYPL we noticed a lot of patrons with multiple barcodes, some of which looked like like "12345678901234LOST". This indicates that their barcode _used_ to be 12345678901234, but they lost their library card, and when they were issued a new one the librarian stuck "LOST" on the end of their old barcode instead of deleting it from the system. So we introduced `authorization_identifier_blacklist`. It helps us see when a piece of data in the barcode field is actually a note about a prior barcode, not an active barcode.