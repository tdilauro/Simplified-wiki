To support multiple libraries on a single circulation manager, we need to allow a single circulation manager to be configured with several different authentication mechanisms. We already have a configurable set of authentication mechanisms, but configuration itself happens within the JSON file. In a multi-library environment this configuration needs to happen through the web, and the configuration needs to be stored in the database.

Currently AuthenticationProviders are instantiated from site config. In the future they will be instantiated from the database.

Here are the current AuthenticationProviders along with the values you need to configure them.

# `SIPAuthenticationProvider`

Found in `api/sip/__init__.py`.

* server
* port
* login_user_id
* login_password
* location_code
* field_separator

# `MilleniumPatronAPI`

Found in `api/millenium_patron.py`.

* url
* authorization_identifier_blacklist (a list)
* verify_certificate (a boolean)

# `FirstBookAuthenticationAPI`

Found in `api/firstbook.py`

* url
* key (an API key)

# `CleverAuthenticationAPI`

Found in `api/clever/__init__.py`

* client_id
* client_secret
* token_expiration_days

# `OAuthAuthenticationProvider`

This is an abstract class. There is currently no "generic OAuth" provider. To add an OAuth provider you must define a subclass in Python code, a la `CleverAuthenticationAPI`, and define values for the constants URI, METHOD (optional), NAME, TOKEN_TYPE, TOKEN_DATA_SOURCE_NAME, and EXTERNAL_AUTHENTICATE_URL. You need to implement `remote_exchange_authorization_code_for_access_token` and `remote_patron_lookup`. And _then_ you need to provide `client_id`, `client_secret`, and `token_expiration_days` in the constructor.

It might not be necessary to do this in all cases. It might be possible to configure `OAuthAuthenticationProvider` with an `ExternalIntegration` that contains values for those variables. Then we could implement a default `remote_exchange_authorization_code_for_access_token` that works in common cases, and implement `remote_patron_lookup` as a no-op.