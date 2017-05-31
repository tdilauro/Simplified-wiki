To support multiple libraries on a single circulation manager, we need to allow a single circulation manager to be configured with several different authentication mechanisms. We already have a configurable set of authentication mechanisms, but configuration itself happens within the JSON file. In a multi-library environment this configuration needs to happen through the web, and the configuration needs to be stored in the database.

Currently AuthenticationProviders are instantiated from site config. After this work is done, they will be instantiated from the database.

# What we know

After sending out a lot of questionnaires, we now know a few things about how different libraries authenticate patrons.

* NYPL authenticates all patrons through Millenium Patron.
* Open Ebooks can authenticate patrons through Clever or through FirstBook.
* The Ferguson, CT library authenticates all patrons through SIP2. Only patrons whose barcodes start with 2111800 have borrowing privileges.
* The LION consortium authenticates all patrons through Millenium Patron. The value of a patron's CCARD[p46] field determine the patron's home library within the consortium.
* The LCI consortium authenticates all patrons through Millenium Patron. The first five characters of a patron's barcode determine the patron's home library within the consortium.
* The Bibliomation consortium authenticates all patrons through SIP2. The value of the AQ field determines the patron's home library within the consortium.

We also know a few things about how different libraries _authorize_ patrons. Not all patrons have borrowing privileges, and not all patrons with borrowing privileges can borrow all books.

* In general, patrons lose borrowing privileges if they are blocked or if they accrue excessive fines.
* Although all Sierra libraries use the `MBLOCK[p56]` field to convey block status, different ILS installations have different rules about which values for `MBLOCK[p56]` mean the patron has lost borrowing privileges.
* Rules about what dollar amount constitutes an "excessive" fine seem to be set ILS-wide, meaning that different libraries within a consortium do not have latitude to change this number.
* NYPL uses the patron type (derived from the `P TYPE[p47]` field) to control access to different types of books. For example, your patron type may restrict you to borrowing childrens' books.
* Open Ebooks uses the patron type (derived from the FirstBook barcode for or by asking the Clever API for grade level) to control which lane you're directed to. For example, your patron type may send you to the Early Grade lane or the Young Adult lane.

# Existing authentication providers

Here are the current AuthenticationProviders along with the values you need to configure them.

In addition to the values specified, all Basic Auth authentication providers can be configured with these four options:

* identifier_regular_expression
* password_regular_expression
* test_username
* test_password

## `SIPAuthenticationProvider`

Found in `api/sip/__init__.py`.

* server
* port
* login_user_id
* login_password
* location_code
* field_separator

## `MilleniumPatronAPI`

Found in `api/millenium_patron.py`.

* url
* authorization_identifier_blacklist (a list)
* verify_certificate (a boolean)

## `FirstBookAuthenticationAPI`

Found in `api/firstbook.py`

* url
* key (an API key)

## `MockAuthenticationProvider`

Found in `api/mock_authentication.py`.

* patrons (a dictionary mapping usernames to passwords)
* expired_patrons (a dictionary mapping usernames to passwords)
* patrons_with_fines (a dictionary mapping usernames to passwords)

This is problematic because its main attraction is that it's the simplest authentication provider to set up. When configuration is done through a JSON file, it is much simpler to write a `patrons` dictionary than to try to get SIP2 working. But when configuration is done through a web interface that expects a relatively small number of scalar configuration values, it becomes a lot less simple (relatively speaking) to input this kind of configuration.

In the interests of simplicity it might be better to allow MockAuthenticationProvider to accept only a single username and password (the `test_username` and `test_password`).

## `CleverAuthenticationAPI`

Found in `api/clever/__init__.py`

* client_id
* client_secret
* token_expiration_days

## `OAuthAuthenticationProvider`

This is an abstract class. There is currently no "generic OAuth" provider. To add an OAuth provider you must define a subclass in Python code, a la `CleverAuthenticationAPI`, and define values for the constants URI, METHOD (optional), NAME, TOKEN_TYPE, TOKEN_DATA_SOURCE_NAME, and EXTERNAL_AUTHENTICATE_URL. You need to implement `remote_exchange_authorization_code_for_access_token` and `remote_patron_lookup`. And _then_ you need to provide `client_id`, `client_secret`, and `token_expiration_days` in the constructor.

It might not be necessary to do this in all cases. It might be possible to configure `OAuthAuthenticationProvider` with an `ExternalIntegration` that contains values for those variables. Then we could implement a default `remote_exchange_authorization_code_for_access_token` that works in common cases, and implement `remote_patron_lookup` as a no-op.

# Database schema changes

The `externalintegrations` table will be used to store all configuration currently stored in JSON configuration. This includes configuration items like `test_username` and `test_password`, which will be necessary to run a self-test from the administrative interface. 

(In theory, different libraries that use the same ILS might provide different test patrons, but in practice we generally get a single test patron. So I'm going to say the purpose of test_username and test_password is to verify that the connection to the ILS is working and we can speak its language, rather than to verify that each library has configured the ILS correctly.)

An `ExternalIntegration` for an authentication mechanism has `goal=PATRON_AUTH_GOAL` and a `protocol` corresponding to the authentication technique (SIP2, Millenium Patron, Clever, etc).

When there's only one library, we can use the same code we have now, but pull the AuthenticationProvider configuration from a database query rather than from the JSON config. However, when there's more than one library, different libraries may use different authentication mechanisms. Libraries that use the same authentication mechanism need to determine whether a patron who passes ILS authentication is actually a patron of _that_ library, rather than a different library on the same ILS.

So after moving authentication configuration to `ExternalIntegration, we will create a new `patronauthenticationservices` table, by analogy to `adminauthenticationservices`, that looks like this:

```
patronauthenticationservices
 id
 library_id
 patron_restriction_type
 patron_restriction
 external_integration_id
```

This table creates a many-to-many relationship between libraries and patron authentication services. Most libraries will only have one authentication service, but some (Open Ebooks) will have more than one.

If a library will accept anyone who the `ExternalIntegration` thinks is okay, then `patron_restriction_type` and `patron_restriction` can be left empty. If a library needs to distinguish between patrons of _this_ library (who get service) and patrons of _some other_ library (who don't), then `patron_restriction_type` and `patron_restriction` should be set to appropriate values.

Appropriate values for `patron_restriction_type` and `patron_restriction` might be "identifier prefix"/"23333" or "library code"/"56". Once patron information is obtained from the ILS and put into a generic form, the patron restriction type would be imposed to grant or deny access.

# Admin interface

We need interfaces for doing the following:

* List patron authentication services
* Create a new patron authentication service
* Configure an existing patron authentication service
* Run a self-test on a patron authentication service (using the test username and password configured for that service).
* Associate a patron authentication service with a library, possibly adding a patron restriction
* Modify or remove the patron restriction on a library+patron authentication service
* Disassociate a patron authentication service from a library
* Delete a patron authentication service not used by any libraries