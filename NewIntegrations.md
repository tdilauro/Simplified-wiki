The goal of the Library Simplified circulation manager is to tie
together many pieces of software: ILS systems for authenticating
patrons, license sources for borrowing books, search interfaces for
finding books, analytics tools for reporting on usage, and so on.

When you're seting up a circulation manager for a library, all the
currently supported integrations can be configured through the
administrative interface. When you're adding _a brand new
integration_, you'll need to go down a level and write some code.

# New patron authentication technique

You probably don't need to write a brand new module to connect the
circulation manager to your ILS. The circulation manager only needs to
communicate with your ILS to verify patron credentials and to get
basic information about their accounts -- fine amounts, card
expiration dates, maybe an email address. You can do this through
SIP2, and most ILS systems already support SIP2.

If you do need to write a new authentication technique, you
can; you just need to write a subclass of `AuthenticationProvider`.

It's more likely that you'll have to write some code because your
library uses an already supported ILS in an unusual way. We'll handle
this on a case-by-case basis, but it probably means you'll need to add
a configuration option to an existing `AuthenticationProvider`
subclass.

## Where to put the code

Put your code in its own module in the `circulation` repository. The module must define a class called `AuthenticationProvider`. If you look at any of the existing authentication modules such as [the SIP2 module](https://github.com/NYPL-Simplified/circulation/blob/master/api/sip/__init__.py) you'll see that they define an authentication provider class and then assign it to `AuthenticationProvider` at the last line of the module.

To get your authentication provider to show up in the admin interface you'll need to add it to [the admin interface's controller.py](https://github.com/NYPL-Simplified/circulation/blob/master/api/admin/controller.py). The `patron_auth_services` method defines a list of `provider_apis` and a list of `basic_auth_protocols`; you'll need to add your `AuthenticationProvider` subclass to the first list and its `.__module__` to the second list. (TODO: this should be put in a place that's easier to edit.)

## `PatronData`

The `PatronData` class, located in [authenticator.py](https://github.com/NYPL-Simplified/circulation/blob/master/api/authenticator.py),
is an ILS-independent abstraction of the information a library keeps
about a patron. Any time you subclass `AuthenticationProvider`, your
job is basically to talk to the ILS and turn what it says about a
patron into a `PatronData` object.

* `permanent_id`: A unique and unchanging identifier for the patron, as
  used by the account management system and probably never seen by the
  patron. This is not required, but it is very useful to have, because
  other identifiers tend to change.

* `authorization_identifier`: One or more assigned identifiers (usually
  numeric) the patron may use to identify themselves. In most
  libraries this is called a "barcode" or a "library card
  number".

  Some libraries allow patrons to have multiple authorization
  identifiers. For example, a patron of NYPL may have an NYPL library
  card, a Brooklyn Public Library card, and an IDNYC card in their ILS
  record. That's three different "barcodes" that all authenticate the
  same patron. A library like this may use a _list_ of identifiers as
  `authorization_identifier`. Any identifier in the list will work.

  When a patron has multiple authorization identifiers, the
  circulation manager does the best it can to maintain continuity of
  the patron's identity in the face of changes to this list. The two
  assumptions made are:

  1) A patron tends to pick one of their authorization
  identifiers and stick with it until it stops working, rather
  than switching back and forth between them.

  2) In the absence of any other information, the authorization
  identifier at the _beginning_ of the `authorization_identifier` list
  is the one the patron is most likely to use.

* `username`: An alphanumeric identifier that identifies the patron,
   generally chosen by the patron themselves.

* `personal_name`: The patron's personal name, provided by them to the
  library. This is not stored in the circulation manager database,
  because we treat it as personally identifiable information, but it
  may be passed on to the client immediately after being retrieved
  from the ILS.

* `email_address`: The patron's email address, provided by them to the
   library. This is not stored in the database, because we treat it as
   personally identifying information. It's used to set up an email
   notification when a patron places a book on hold.

* `authorization_expires`: The date, if any, at which the patron's
  authorization to borrow items from the library expires. This value
  should be a `datetime` object. If not set, it means the patron's
  authorization will never expire.

* `external_type`: A string classifying the patron according to some
  library-specific scheme, often called a _patron type_. By itself, a
  patron's `external_type` has no effect on their access to the
  library. However, other configuration settings may restrict access
  to patrons with certain `external_type`s.

* `fines`: A `Money` object representing the amount the patron owes in
   fines.

* `block_reason`: This field contains the reason why a patron was
   manually blocked from accessing the library's resources (as opposed
   to being automatically blocked due to an expired library card or
   unpaid fines).
   1. `PatronData.NO_VALUE`: The patron is not blocked.
   2. `PatronData.UNKNOWN_REASON`: The patron is blocked but the precise reason is not important.
   3. `PatronData.CARD_REPORTED_LOST`: The patron's library card is reported as lost and they have not been given 
       a new card.
   4. `PatronData.EXCESSIVE_FINES`: The patron is blocked due to excess unpaid fines. This may be true even if their 
       _current_ level of fines is not available from the ILS or does not exceed the library's maximum fine amount.

* `complete`: A boolean indicating whether or not this `PatronData` includes all the information you can reasonably expect to get about this patron from the ILS. As you'll see below, if you can return a complete `PatronData` from `BasicAuthenticator.remote_authenticate()`, you can make your integration a lot more efficient.



## Subclassing `BasicAuthenticationProvider`

The most common type of library authentication is one where the patron
provides two pieces of information that correspond to a username and
password. The circulation manager uses HTTP Basic Auth to handle this
case.

Within the circulation manager code, `SIP2AuthenticationProvider` and
`MilleniumPatronAPI` are subclasses of
`BasicAuthenticationProvider`. `SimpleAuthenticationProvider` is
another subclass that is intended for testing: instead of validating
patrons against an external ILS, it validates patrons against its own
internal list. It's a good one to learn from.

To subclass `BasicAuthenticationProvider` you'll need to customize a
few attributes and implement a few methods.

### Attributes

There are a number of class attributes you can customize in your
`BasicAuthenticationProvider` subclass. These are the most important:

* `NAME`: The name of the ILS or other system you're integrating
  with. This is displayed in the administrative interface and used to
  distinguish your new integration from other integrations like
  SIP2. _You must provide a value for this attribute._

* `DESCRIPTION`: A human-readable description displayed in the
  administrative interface. This will help administrators understand
  if yours is the integration that will connect their circulation
  manager to their ILS. A value for this attribute is optional but
  recommended.

### `remote_authenticate(self, username, password)`

This is the core of `BasicAuthenticationProvider`. You're given a
username and a password. You need to check whether that combination
identifies a patron. Once you do this work, you have three options:

* Return `True`. This says that the username and password are valid,
  but that you don't know anything else about the patron. If the
  circulation manager needs to check any information about the patron,
  it will need to call `remote_patron_lookup`.
* Return `False`. This says that the username and password do not identify
  a patron on the system.
* Return a `PatronData` object. This says that the username and
  password are valid, _and_ that while you were looking them up, you
  were able to find some information about the patron's account.

If you're able to return a `PatronData` object with `complete=True`,
then that means we'll never have to call `remote_patron_lookup`, and
you won't have to implement it. The `SIP2AuthenticationProvider` is
able to do this, because in SIP2 the API call to check a patron's
credentials also provides complete information about the patron's
account.

### `remote_patron_lookup(self, patron_or_patrondata)`

This method takes incomplete information about a patron, represented
by a database `Patron` object or an abstract `PatronData` object, and
fills it out. It's supposed to return a `PatronData` that contains all
relevant and available information about the patron.

See the `MilleniumPatronAPI` for an example of this. Its
`remote_authenticate` method returns a yes-or-no answer, so unlike the
`SIP2AuthenticationProvider` it can't get away with not implementing
`remote_patron_lookup`. Instead,
`MilleniumPatronAPI.remote_patron_lookup` makes an API call to get all
available information about the patron it was given, parses the
HTML result, and creates a `PatronData` object out of it.

## Subclassing `OAuthAuthenticationProvider`

The other authentication technique used by ILS systems is based on
OAuth. Instead of entering their credentials directly into a mobile
app, the patron is sent to a website controlled by the library and
asked to log in there.

Subclassing `OauthAuthenticationProvider` is not covered yet. I'll
fill in this section as necessary. For an example, see
[CleverAuthenticationAPI](https://github.com/NYPL-Simplified/circulation/blob/master/api/clever/__init__.py).
