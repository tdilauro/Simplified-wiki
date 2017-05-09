To support multiple libraries on a single circulation manager, we need to allow a single circulation manager to be configured with several different authentication mechanisms. We already have a configurable set of authentication mechanisms, but configuration itself happens within the JSON file. In a multi-library environment this configuration needs to happen through the web, and the configuration needs to be stored in the database.

Currently AuthenticationProviders are instantiated from site config. In the future they will be instantiated from the database.

# Existing authentication providers

Here are the current AuthenticationProviders along with the values you need to configure them.

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

In the interests of simplicity it might be better to allow MockAuthenticationProvider to accept only a single username and password (the one designated as the test username/password).

## `CleverAuthenticationAPI`

Found in `api/clever/__init__.py`

* client_id
* client_secret
* token_expiration_days

## `OAuthAuthenticationProvider`

This is an abstract class. There is currently no "generic OAuth" provider. To add an OAuth provider you must define a subclass in Python code, a la `CleverAuthenticationAPI`, and define values for the constants URI, METHOD (optional), NAME, TOKEN_TYPE, TOKEN_DATA_SOURCE_NAME, and EXTERNAL_AUTHENTICATE_URL. You need to implement `remote_exchange_authorization_code_for_access_token` and `remote_patron_lookup`. And _then_ you need to provide `client_id`, `client_secret`, and `token_expiration_days` in the constructor.

It might not be necessary to do this in all cases. It might be possible to configure `OAuthAuthenticationProvider` with an `ExternalIntegration` that contains values for those variables. Then we could implement a default `remote_exchange_authorization_code_for_access_token` that works in common cases, and implement `remote_patron_lookup` as a no-op.

# Database schema changes

Create a new `patronauthenticationservices` table, by analogy to `adminauthenticationservices`, that looks like this:

```
patronauthenticationservices
 id
 name
 protocol
 library_id
 external_integration_id
```

All configuration items will go into the `ExternalIntegration`. This includes configuration items like `test_username` and `test_password`, which will be necessary to run a self-test from the administrative interface.

The model class will enforce rules like "at most one Basic-type authentication mechanism per library".

In general, I think a one-to-many relationship between Library and PatronAuthenticationService is best. In general, each library authenticates its patrons in a distinctive way that no other library may use. That's why I gave this table a library_id. However, there are two cases I can think of when there's a many-to-one or many-to-many relationship between a library and the thing that authenticates the patrons.

In the first case, every library in a consortium authenticates against the same SIP2 server, but there's some special field (such as location_code) which distinguishes between the two libraries. In this case it would be nice to avoid defining thirty slightly different ExternalIntegrations for the same SIP2 server. PatronAuthenticationService could then become a join table between Library and ExternalIntegration which specifies some extra information.

```
patronauthenticationservices
 id
 name
 protocol
 library_id
 external_integration_id
 extra_config
```

It would be annoying to have to come up with a distinct name for each PatronAuthenticationService, but you wouldn't have to specify the SIP2 server multiple times.

In the second case, a number of libraries all authenticate their patrons through the same OAuth server, and you are asked to select your library when you open up the OAuth server in a web view. In this case the ExternalIntegration for every library would be exactly the same, and the PatronAuthenticationServices would also be exactly the same. 

In this case the best database schema might look like this:

```
libraries_patronauthenticationservices
 id
 library_id
 patronauthenticationservice_id
```

```
patronauthenticationservices
 id
 name
 protocol
 external_integration_id
```

Or we could support both cases with a schema like this:

```
libraries_patronauthenticationservices
 id
 library_id
 patronauthenticationservice_id
```

```
patronauthenticationservices
 id
 name
 protocol
 external_integration_id
 extra_config
```

In these cases I'm not convinced that `test_username` and `test_password` belong in the ExternalIntegration, since different libraries probably have different test patrons. I feel closer to saying this stuff should go in `extra_config`.

I think we should make this decision on the basis of usability, both in terms of what we can turn into a simple, consistent interface, and in terms of not making administrators do duplicate work. But those two goals seem in conflict so I don't know what the answer is.

It might be good enough to offer in the admin interface the ability to copy a patron authentication service from one library to another.

# Admin interface

We need interfaces for doing the following:

* List patron authentication services for a library
* Create a new patron authentication service for a library
* Delete a patron authentication service
* Run a self-test on a patron authentication service (using the test username and password configured for that service).


If library-patronauthenticationservice is many-to-many then this list will change in predictable ways (creating an authentication service and associating it with a library will be a separate step).