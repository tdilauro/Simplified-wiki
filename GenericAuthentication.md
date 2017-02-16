# Introduction

A Library Simplified circulation manager may be called upon to authenticate patrons using any of a number of protocols. Some of these protocols are industry standards, like SIP2 and NCIP. Some are ILS-specific, like the Millenium Patron API used by the Sierra ILS. Some are based on OAuth, like the Clever authentication service used by Open Ebooks. Some are one-off product-specific APIs, like the First Book authentication service (also used by Open Ebooks).

The [[authentication setup|AuthenticationSetup]] page explains how to connect your ILS to the currently supported authentication mechanisms. But what if you need to create a brand new authentication mechanism? This page explains the circulation manager's authentication framework so you can get this done with a minimum of new code.

All the classes mentioned in this page can be found in the  module.

# Two questions

To integrate a new authentication technique into the circulation manager, we must be able to answer two questions:

1. Are the provided credentials valid? Do they authenticate a real library patron, or are they incorrect or junk?
2. What relevant information does the library have about a given patron? (Personal name, fines, expiration date, etc.)

Every class that answers these questions must subclass the `AuthenticationProvider` class, found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]. (You're actually more likely to subclass one of its two subclasses: `BasicAuthenticationProvider` or `OAuthAuthenticationProvider`). `AuthenticationProvider` defines two methods which must be implemented differently for every authentication technique:

1. `AuthenticationProvider.authenticated_patron` takes a set of credentials as input and, if the source of truth says they correspond to a real patron, creates a `PatronData` object containing all the information known about the patron.
2. `AuthenticationProvider.remote_patron_lookup` takes a `Patron` object and updates it with fresh information from the source of truth.

# Two core mechanisms

Library Simplified's web and mobile clients can send credentials to the circulation manager in two different ways: using HTTP Basic Auth or using OAuth.

You can see these two core mechanisms in action by going to the [[Open Ebooks online catalog|https://catalog.openebooks.us/]] and trying to borrow a book. You will be given two authentication options: First Book (which uses HTTP Basic Auth) and Clever (which authorizes an OAuth Bearer Token). If you choose First Book, you'll be asked for a barcode and PIN, which will be sent to the circulation manager through HTTP Basic Auth. If you choose Clever, you'll be sent to the Clever website and asked to authenticate with Clever, a necessary step towards getting an OAuth bearer token.

## Basic authentication

The Library Simplified client might ask a patron for a username/password or barcode/PIN combination. The patron's credentials will be sent to the circulation manager using HTTP Basic Auth. On the server side, the incoming request will be handled by a subclass of the `BasicAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

If you subclass `BasicAuthenticationProvider`, you will need to implement the `remote_authenticate` method, which asks the source of truth about a username/password combination, and returns True or False.

You will probably also need to implement `remote_patron_lookup`, since the default implementation is a no-op that assumes the source of truth has no special information about a patron other than the ability to validate username/password.

## OAuth authentication

A Library Simplified client might work with the circulation manager to guide a patron through the process of authenticating with an external authority. The circulation manager will then give the client an OAuth bearer token, which it can use to make authenticated requests to the circulation manager. 

On the server side, the incoming bearer token will be handled by a subclass of the `OAuthAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

If you subclass `OAuthAuthenticationProvider`, you will need to define a number of constants and implement the `oauth_callback` method, which communicates with an OAuth authorization server to exchange an authorization code for an access token. You should not have to implement the `authenticate` method yourself.

You will probably also need to implement `remote_patron_lookup`, since the default implementation is a no-op that assumes the source of truth has no special information about a patron other than the ability to validate username/password.

# `PatronData`

The `PatronData` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]) is an abstract container for information about a patron. In any given situation, you can fill it up with as much or as little information as is available.

The `remote_authenticate` and `remote_patron_lookup` methods both return a `PatronData` object.

For detailed information on what the attributes of `PatronData` mean, see the docstring for `PatronData.__init__`,  but here's a sampling of the information you can provide if it's available.

* The patron's authorization identifiers (e.g. their library barcode or barcodes).
* The patron's name.
* The patron's email address.
* The date at which the patron's library card expires.
* The amount of money the patron owes in fines.

The only requirement is that each patron must have at least one authorization identifier.

# Specific examples

## First Book authentication

Probably the simplest concrete AuthenticationProvider is the one for First Book
(found in [[firstbook.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/firstbook.py]]). `FirstBookAuthenticationProvider` is a `BasicAuthenticationProvider` subclass which does nothing but give a yes-or-no answer. We know absolutely nothing about a First Book patron other than their barcode (a meaningless string that looks like "MQ1GITI96R").

The `remote_authenticate` method takes an incoming barcode and PIN and validates them using an HTTP request to a First Book API. If validation succeeds, the method creates a `PatronData` object using the only information we have available: the barcode. If it fails, the method returns `None`, indicating that the authentication attempt has failed.

`FirstBookAuthenticationProvider` does not implement `remote_patron_lookup`. Since First Book doesn't provide any personal information about patrons, and concepts like fees and expiration dates don't apply, it's fine to leave it as a no-op.

## Mock authentication

In [[mock_authentication.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/mock_authentication.py]] you can find an `AuthenticationProvider` that's a little more complex, but easier to actually set up and use. `BasicAuthenticationProvider` subclass which  takes a dictionary mapping usernames to passwords.

```
provider = MockAuthenticationProvider(patrons={"user1": "password1"})
```

The `remote_authenticate` implementation checks that dictionary. If the username/password combination isn't found, it denies access. Otherwise, it creates a `PatronData` object for the patron and returns it. Sometimes it puts some extra information in the `PatronData` object like an expiration date or some fines--you can check the implementation for details.

`MockAuthenticationProvider` does not implement `remote_patron_lookup`, even though the "provider" sometimes knows extra information about a patron (expiration date and fines), because that information is obtained automatically as a side effect of authentication. There's no need for a separate authentication step.

The `SIP2AuthenticationProvider` (found in [[sip/__init__.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/sip/__init__.py]]) does the same thing. When you authenticate a patron through SIP2, the response message you get includes information about the patron. This means we can automatically update patron information every time we authenticate them, without having to make a separate "tell me about this patron" request.

## Millenium Patron authentication

The `MilleniumPatronAPI` is an example of an `AuthenticationProvider` that implements both `remote_authenticate` and `remote_patron_lookup`. It needs to do this because the Millenium Patron API has different endpoints for validating patron credentials ("pintest") and retrieving a patron record. ("dump").

## Clever OAuth authentication

The `CleverAuthenticationProvider` can be found in [[clever/__init__.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/clever/__init__.py]]. It defines a number of constants: `URI`, `NAME`, `TOKEN_TYPE`, `TOKEN_DATA_SOURCE_NAME`, and `EXTERNAL_AUTHENTICATE_URL`. The meanings of these are specified in the comment at the top of the `OAuthAuthenticationProvider` class. For example, `EXTERNAL_AUTHENTICATE_URL` is a Python string template used to generate the clever.com URL that will demand a patron's Clever credentials.

Once the patron logs in to Clever, Clever sends them back to the circulation manager with a verification code. The `oauth_callback` method is triggered with this code as an argument. The job of `oauth_callback` is to communicate with Clever and convert that code into an OAuth bearer token.

Now that we have an OAuth bearer token, we can call `remote_patron_lookup` and find out details about this patron, like their name and the school they attend.