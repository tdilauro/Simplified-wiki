# Overdrive integration

This document is to help you connect your library's Overdrive
collection to your Library Simplified circulation manager.

First, you should know that Overdrive provides a
[glossary](https://developer.overdrive.com/docs/reference-guide) that
explains most of the terminology.

## Getting access

To start connecting your library's Overdrive account to your Library
Simplified circulation manager, go to the [Member
Center](https://developer.overdrive.com/member-center) and apply for
API access. You will probably need to create an Overdrive developer
acount.

You'll be asked which API (actually which authentication technique)
you want access to. Ask for access to all three. You will probably
only need two of the three, but the approval process is slow, it
doesn't cost anything extra to get all three, and you won't have to go
back and re-apply if your authentication situation changes.

Under "Planned API usage", mention the name of your library and that
you are integrating your Overdrive collection into the SimplyE system.

It can take a week or more to get approved for API access, so don't
put this off!

When Overdrive gets back to you, they may give you a website ID of 100300 and a library ID of 4425. Those are the IDs for the Overdrive test library. If this happens to you, go back to them and tell them that you need the production IDs to integrate your library into the SimplyE system.

## Types of APIs

Overdrive distinguishes between "Discovery" and "Circulation" APIs. To
quote the glossary:

_Discovery APIs_: Discovery APIs are designed to allow your users to
 browse and explore OverDrive digital collections. You can search for
 titles, check availability, and get details on specific titles.

_Circulation APIs_: These APIs are designed to allow you to circulate
 content from an OverDrive digital collection. You can borrow and
 place holds on titles, see what a specific user has borrowed or
 placed on hold, and get download links for content that a user has
 borrowed.

Library Simplified uses the Discovery APIs to keep track of the items
in your Overdrive collection, and the Circulation APIs to conduct
transactions on behalf of your patrons. Library Simplified needs
access to both.

## Types of authentication

When the Library Simplified circulation manager makes a call to the
Overdrive API, Overdrive needs to verify that the circulation manager
is authorized to act on behalf of your library. If the circulation
manager is acting on behalf of a specific patron (e.g. creating a
loan), Overdrive also needs to know that the circulation manager is
authorized to act on behalf of that patron.

Overdrive offers three types of authentication:

[Client
authentication](https://developer.overdrive.com/apis/client-auth)
verifies that the circulation manager is authorized to act on behalf
of the library. It's used by scripts that run in the background to
maintain an accurate picture of your library's Overdrive collection.

[Patron
authentication](https://developer.dev.overdrive.com/apis/patron-auth)
and [Granted
authentication](https://developer.dev.overdrive.com/granted-auth) are
used when the circulation manager is carrying out the wishes of a
specific patron.

In all three cases, the goal is to get an _access token_. An access
token is a string that you can present to the Overdrive API to get it
to actually do something. An Overdrive access token is generally good for one
hour. Once it expires, you need to get a new one.

### Client authentication

[Client
authentication](https://developer.overdrive.com/apis/client-auth) is
pretty simple. Overdrive issues you a set of credentials: a client key
and a client secret. At any time, you can show those credentials to
Overdrive to get an access token.

Although this is the simplest way to get an an access token, the token
is not authorized to act on behalf of a specific patron. You can look
at the collection but you can't borrow books.

### Patron authentication

[Patron
authentication](https://developer.dev.overdrive.com/apis/patron-auth)
is the simplest way of getting an access token to act on behalf of a
patron. It can only be used when your library does authentication by
username and password (or equivalent pieces of information, such as
barcode and PIN).

To get an access token with patron authentication, you provide your
client key and client secret (as with client authentication), but you
also provide the patron's username and password.

Overdrive checks the patron's username and password with your ILS. If
the ILS says the username and password are valid, you get an access token.

This access token can act on behalf of the patron whose username it
was associated with. You can borrow books, place holds, etc.

As with other access tokens, this token is only good for an hour.

### Granted authentication

[Granted authentication](https://developer.overdrive.com/granted-auth)
is a more complex way of getting an access token to act on behalf of a
patron.

When you set up granted authentication with Overdrive, you'll be asked
to set up a "redirect URI". (You can change your redirect URI
[here](https://developer.overdrive.com/member-center/edit-auth-fields).)
Your redirect URI should point to the `oauth_calback` controller of
your circulation manager, e.g.:

```
https://my-circulation-manager.com/oauth_callback?provider=Overdrive
```

When a patron tries to check out a book through Overdrive, Library
Simplified circulation manager will tell them to visit a URL based on
this template:

```
https://oauth.overdrive.com/auth?client_id={ID}&redirect_uri={URLredirectedTo}&scope=accountId:{ID}&response_type=code&state={optionalStateParameter}
```

Overdrive will redirect the patron to... some URL somewhere. I'm not sure yet if it's a URL on your ILS or one managed by Overdrive. Either way, the patron will be
asked to log in. Once they log in they will be asked to authorize
Library Simplified to act on their behalf. Once they allow this,
Overdrive will send them to your redirect URI:

```
https://my-circulation-manager.com/oauth_callback?provider=Overdrive&state={optionalStateParameter}&code={authorizationCode}
```

The circulation manager can then use the `{authorizationCode}` to get
an access token to act on behalf of the patron.
