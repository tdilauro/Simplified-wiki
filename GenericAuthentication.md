# Introduction

A Library Simplified circulation manager may be called upon to authenticate patrons using any of a number of protocols. Some of these protocols are industry standards, like SIP2 and NCIP. Some are ILS-specific, like the Millenium Patron API used by the Sierra ILS. Some are based on OAuth, like the Clever authentication service used by Open Ebooks. Some are one-off product-specific APIs, like the FirstBook authentication service (also used by Open Ebooks).

The [[authentication setup|AuthenticationSetup]] page explains how to connect your ILS to the currently supported authentication mechanisms. But what if you need to create a brand new authentication mechanism? This page explains the circulation manager's authentication framework so you can get this done with a minimum of new code.

All the classes mentioned in this page can be found in the  module.

# Two questions

To integrate a new authentication technique into the circulation manager, we must be able to answer two questions:

1. Are the provided credentials valid? Do they authenticate a real library patron, or are they incorrect or junk?
2. What relevant information does the library have about a given patron? (Personal name, fines, expiration date, etc.)

Every class that answers these questions must subclass the `AuthenticationProvider` class, found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]. It defines two methods which must be implemented differently for every authentication technique:

1. `AuthenticationProvider.authenticated_patron` takes a set of credentials as input and, if the source of truth says they correspond to a real patron, finds or creates the corresponding `Patron` object. (`Patron` is a database object representing the circulation manager's view of a library patron.)
2. `AuthenticationProvider.remote_patron_lookup` takes a `Patron` object and updates it with fresh information from the source of truth.

# Two core mechanisms

Library Simplified's web and mobile clients can send credentials to the circulation manager in two different ways: using HTTP Basic Auth or using OAuth.

You can see these two core mechanisms in action by going to the [[Open Ebooks online catalog|https://catalog.openebooks.us/]] and trying to borrow a book. You will be given two authentication options: FirstBook (which uses HTTP Basic Auth) and Clever (which authorizes an OAuth Bearer Token). If you choose FirstBook, you'll be asked for a barcode and PIN, which will be sent to the circulation manager through HTTP Basic Auth. If you choose Clever, you'll be sent to the Clever website and asked to authenticate with Clever, a necessary step towards getting an OAuth bearer token.

## Basic authentication

The Library Simplified client might ask a patron for a username/password or barcode/PIN combination. The patron's credentials will be sent to the circulation manager using HTTP Basic Auth. On the server side, the incoming request will be handled by a subclass of the `BasicAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

If you subclass `BasicAuthenticationProvider`, you will need to implement the `remote_authenticate` method, which asks the source of truth about a username/password combination, and returns True or False.

You will probably also need to implement `remote_patron_lookup`, since the default implementation is a no-op that assumes the source of truth has no special information about a patron other than the ability to validate username/password.

## OAuth authentication

A Library Simplified client might work with the circulation manager to guide a patron through the process of authenticating with an external authority. The circulation manager will then give the client an OAuth bearer token, which it can use to make authenticated requests to the circulation manager. 

On the server side, the incoming bearer token will be handled by a subclass of the `OAuthAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

If you subclass `OAuthAuthenticationProvider`, you will need to define a number of constants and implement the `oauth_callback` method, which communicates with an OAuth authorization server to exchange an authorization code for an access token.

You will probably also need to implement `remote_patron_lookup`, since the default implementation is a no-op that assumes the source of truth has no special information about a patron other than the ability to validate username/password.

# Specific examples

## Mock authentication

The simplest `AuthenticationProvider` can be found in [[mock_authentication.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/mock_authentication.py]]. This provider's constructor takes a dictionary mapping usernames to passwords.

```
provider = MockAuthenticationProvider(patrons={"user1": "password1"})
```

