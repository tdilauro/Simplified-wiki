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

Library Simplified's web and mobile clients can send credentials in two different ways. First, they might ask a patron for a username/password or barcode/PIN combination. The patron's credentials will be sent to the circulation manager using HTTP Basic Auth. On the server side, the incoming request will be handled by a subclass of the `BasicAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

Second, a Library Simplified client might guide a patron through the process of authenticating with an external authority. The client will then be given an OAuth bearer token which it can send when making authenticated requests to the circulation manager. On the server side, the incoming bearer token will be handled by a subclass of the `OAuthAuthenticationProvider` class (found in [[authenticator.py|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]]).

You can see these two core mechanisms in action by going to the [[Open Ebooks online catalog|https://catalog.openebooks.us/]] and trying to borrow a book. You will be given two authentication options: FirstBook (which uses HTTP Basic Auth) and Clever (which authorizes an OAuth Bearer Token). If you choose FirstBook, you'll be asked for a barcode and PIN, which will be sent to the circulation manager through HTTP Basic Auth. If you choose Clever, you'll be sent to the Clever website and asked to authenticate with Clever, a necessary step towards getting an OAuth bearer token.

