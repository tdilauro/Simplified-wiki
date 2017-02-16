# Introduction

A Library Simplified circulation manager may be called upon to authenticate patrons using any of a number of protocols. Some of these protocols are industry standards, like SIP2 and NCIP. Some are ILS-specific, like the Millenium Patron API used by the Sierra ILS. Some are based on OAuth, like the Clever authentication service used by Open Ebooks. Some are one-off product-specific APIs, like the FirstBook authentication service (also used by Open Ebooks).

The [[authentication setup|AuthenticationSetup]] page explains how to connect your ILS to the currently supported authentication mechanisms. But what if you need to create a brand new authentication mechanism? This page explains the circulation manager's authentication framework.

All the classes mentioned in this page can be found in the [[`authenticator`|https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py]] module.

# Two questions

To integrate an authentication method into the circulation manager, we must be able to answer two questions:

1. Are the provided credentials valid?
2. What relevant information does the library have about a given patron? (Personal name, fines, expiration date, etc.)

The superclass of all authentication providers is the `AuthenticationProvider` class. It defines two methods which must be 

1. `AuthenticationProvider.authenticated_patron` takes a set of credentials as input and tries to turn them into a `Patron` object. (`Patron` is a database object representing the circulation manager's view of a library patron.)
2. `AuthenticationProvider.remote_patron_lookup` takes a `Patron` object and updates it with fresh information from the source of truth.

# Two core mechanisms

The web and mobile clients for Library Simplified can send credentials in two different ways. First, they might ask a patron for a username/password or barcode/PIN combination. The patron's credentials will be sent to the circulation manager using HTTP Basic Auth. On the server side, the incoming request will be handled by an instance of the `BasicAuthenticationProvider` class.

Second, a Library Simplified client might guide a patron through the process of authenticating with an external authority and authorizing an OAuth Bearer Token. The client will then be given an authorization token which it can send when making authenticated requests to the circulation manager.

You can see these two core mechanisms in action by going to the [[Open Ebooks online catalog|https://catalog.openebooks.us/]] and trying to borrow a book. You will be given two authentication options: FirstBook (which uses HTTP Basic Auth) and Clever (which authorizes an OAuth Bearer Token).