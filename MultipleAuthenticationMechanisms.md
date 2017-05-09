To support multiple libraries on a single circulation manager, we need to allow a single circulation manager to be configured with several different authentication mechanisms. We already have a configurable set of authentication mechanisms, but configuration itself happens within the JSON file. In a multi-library environment this configuration needs to happen through the web, and the configuration needs to be stored in the database.

Currently AuthenticationProviders are instantiated from site config. In the future they will be instantiated from the database.

Here are the current AuthenticationProviders along with their 

# `SIPAuthenticationProvider`

Found in `api/sip/__init__.py`.

* server
* port
* login_user_id
* login_password
* location_code
* field_separator

# `MilleniumPatronAPI`

Found in api/millenium_patron.py.

* url
* authorization_identifier_blacklist (a list)
* verify_certificate (a boolean)

# `CleverAuthenticationAPI`

Found in api/clever/__init__.py

* client_id
* client_secret
* token_expiration_days

There is no "generic OAuth" AuthenticationProvider. Currently adding an OAuth provider that delegates to a different OAuth server requires defining a subclass in Python code, a la `CleverAuthenticationAPI`, and define values for the constants URI, METHOD (optional), NAME, TOKEN_TYPE, TOKEN_DATA_SOURCE_NAME, and EXTERNAL_AUTHENTICATE_URL. However, it might not be necessary to do this. It might be possible to configure `OAuthAuthenticationProvider` with an `ExternalIntegration` that contains values for those variables.