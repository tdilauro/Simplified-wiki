Although any OPDS server can interact with any OPDS client, creating a seamless user experience requires the server to provide a lot of information not covered in the OPDS spec.

A simple example: OPDS doesn't say anything about authentication. Although library patrons are generally authenticated by a mechanism that's compatible with HTTP Basic Auth, that's not always true. Even when it is true, providing a generic UI that asks for "username" and "password" creates a bad experience for a user who is expecting to type in a "library card number" or a "barcode", not a "username".

The [Authentication for OPDS](https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw/edit#heading=h.r2fysm93j6kk) (A4OPDS) spec gives an OPDS server a way to explain how its clients should present the authentication interface.

This document expands the role of the A4OPDS document past simple authentication and into the realm of discovery. In the Library Simplified ecosystem, there are thousands of OPDS servers, each serving a slightly different audience with different content. For a person to navigate all these servers they need access to a directory service. To build a directory service, there needs to be a way for each server to describe not only the process of authenticating to the server, but _why_ someone would want to authenticate in the first place, and what _sort_ of person might be able to authenticate.

We expand the A4OPDS document, rather than creating a new type of document, because the A4OPDS spec defines the special requirements needed by a directory listing document. In particular, this document must always be available without authentication.

With our extensions, the A4OPDS document contains everything a potential user needs to realize they want to access an OPDS server, determine their eligibility, find the server, obtain credentials, and authenticate. Once the user is authenticated, OPDS itself takes over.

# Link to the A4OPDS document from server root

The root feed of an OPDS server should link to its Authentication For OPDS document using `rel="http://opds-spec.org/auth/document"` and a `type` that makes it clear the document on the other end is an A4OPDS document (that is, `type="application/vnd.opds.authentication.v1.0+json"`). This guarantees that a client can always find the A4OPDS document. If the root feed requires authentication, that's fine, because the A4OPDS document is also supposed to be served as the entity-body of a 401 response.

# Server description

An OPDS server may use the `service_description` extension to describe itself. This is distinct from the standard `description` field, which is to be used to describe the text prompt displayed to the authenticating user.

```
 "service_description": "Here you can get all sorts of free books!",
```

# Color scheme

An OPDS server may use the `color_scheme` extension to specify the color scheme a client should use when rendering that server's OPDS feeds.

`"color_scheme": "blue",`

The color schemes supported by SimplyE are "red", "blue", "gray", "gold", "green", "teal", and "purple". The specific colors used by these color schemes are laid out in the [NYPL Design Toolkit](https://nypl.github.io/design-toolkit/sections/color.html). ("Gold" is a nicer name for yellow.)

# Collection size

An OPDS server may use `collection_size` to advertise the number of distinct items of content available to a typical user of its collection.

`collection_size` may be an integer representing the total size of the collection, or it may be a dictionary that maps an [ISO 639-2 Alpha-3 language code](https://en.wikipedia.org/wiki/ISO_639-2) to the total collection size _for that language_. This does not have to include every single language available through the server, but it should include all languages that are easily discoverable through navigating the server's OPDS feeds.

This `collection_size` says that an OPDS server offers about 100,000 items total:

```
"collection_size": 100000,
```

This says that an OPDS server offers ten titles in English, four in Japanese, and one in Chinese:

```
"collection_size": {
 "eng": 10,
 "jpn": 4,
 "chi": 1
},
```

The numbers do not need to be precise measurements, but they should be accurate to within an order of magnitude.

Clients should not draw any conclusions from the absence of a value for `collection_size`, or from the use of a single number rather than a dictionary splitting out the collection by language.

If a server splits out its collections by language, it _is_ fair to assume that languages not mentioned do not have a significant presence on the server, but not to assume that the server has no titles whatsoever in those languages. A server may explicitly indicate that it has no titles in a language by associating it with a value of zero.

# Public key

If your OPDS server needs to receive cryptographically signed messages (e.g. to set up shared secrets with other servers), you can publish your public key in the authentication document.

```
"public_key": { 
  "type": "RSA",
  "value": "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDDbk5XAE+bLeDLQrl9QtkHh2Z2\nQkZ5NE+VesPOP//Z0UjMpI+cDDdC7DstOQZqYlvr9HMsv0syDja9M8aiyyIXcw4v\nrANiK/7MnpbbWII32YhzQ2p48IVobCWgtTBKqjHuvGY5Y8lr/s6xddx8GnueCpeN\n6lnZq+a86CBm5C+/yQIDAQAB\n-----END PUBLIC KEY-----"
}
```
# Input mechanisms

An OPDS server may use the `inputs` extension to customize the input mechanisms used to gather the patron's credentials. 

The value of `inputs` should be a JSON object. Each key in the `inputs` object names a field that's being configured. The only two fields currently defined are `login` and `password.

This is distinct from the standard `labels` field, which allows a server to customize the names displayed for the "login" and "password" fields. Here's an example that uses both:

```
{
 "inputs": {
   "login": {
     "keyboard": "full",
     "maximum_length": 20,
     "barcode_format": "CODABAR"
   },
   "password": {
     "keyboard": "Number pad",
     "maximum_length": 4
   }
 },
 {
  "labels": {
     "login": "Username",
     "password": "Access code",
 }
}
```

We keep them separate to avoid changing the definition of `labels`, a field that's defined in the spec.

## Keyboard

The `keyboard` extension within `inputs` lets you specify the keyboard that should be used to gather the user input for a given field. The following values are defined:

* `Default` - The default keyboard for the platform.
* `Email address` - A keyboard optimized for entering email addresses.
* `Number pad` - A numeric keypad.

## Maximum length

The `maximum_length` extension lets you specify the maximum length of a given input field. Its value is a number.

This extension lets clients size UI elements appropriately. More importantly, tf `maximum_length` is set to zero, the client should omit the corresponding UI element altogether, and send an empty string as the value for the corresponding field.

## Barcode format

Libraries sometimes issue plastic cards that contain machine-readable representations of a patron's login identifier. The `barcode_format` extension lets an OPDS server explain how to turn an image (e.g. from a phone camera) into a value for the login identifier.

The following values are defined for this extension.

* `Codabar`: Try to scan a barcode in [[Codabar|https://en.wikipedia.org/wiki/Codabar]] format.

Unlike other field extensions, the `barcode_format` extension is only defined on the `login` field. It has no effect if defined on the `password` field.

# Feature flags

An OPDS server may wish to enable or disable certain common features of OPDS clients, or to set expectations up front as to what kind of server this is. Often this can be done by providing or omitting links with certain link relations, but when this isn't enough, a server can add a feature flag URI to the `feature_flags` extension.

Note that an OPDS client is not obligated to respect a feature flag--it can provide UI for a "disabled" feature and decide not to show UI for an "enabled" one.

The `features` extension object contains two optional keys, `enabled` and `disabled`. If present, each maps to a list of URIs indicating features that are enabled or disabled for that server.

The following URIs are defined for use as feature flags:

* `https://librarysimplified.org/rel/feature/reservations`: This feature is enabled by default. If it is disabled, a client should not show any indication that it's possible for a user to place a reservation for a title. A title is either available right now or it's not.
* `http://opds-spec.org/acquisition/borrow`: This feature signals that this OPDS server allows users to borrow content for a limited time. Clients should not draw any conclusions from the absence of this feature from the `features` object; instead, they should look in the actual OPDS Feeds and see if links with this relation show up.
* `http://opds-spec.org/acquisition/open-access`: This feature signals that this OPDS server allows users to directly download content, unencumbered by DRM, and keep it permanently. Clients should not draw any conclusions from the absence of this feature from the `features` object; instead, they should look in the actual OPDS Feeds and see if links with this relation show up.
* `http://opds-spec.org/acquisition/buy`: This feature signals that this OPDS server allows users to pay for content and keep it (or access to it) permanently. Clients should not draw any conclusions from the absence of this feature from the `features` object; instead, they should look in the actual OPDS Feeds and see if links with this relation show up.

# Specifying your audience

Different OPDS servers serve people in different ways. These extensions allow OPDS servers to explain on a high level how they operate and which populations they serve.

These fields are purely advisory and, by themselves, have no effect on who can use the OPDS server. Once you connect to the server, you either have the credentials or you don't. The purpose of these fields is to aid discovery. Systems like the Library Simplified library registry use this data to help people distinguish between (for instance) their local public library and a private, members-only library in the same city.

## `audiences`

Some collections are open to the general public; others restrict access to students or people with other special qualifications.

The `audiences` field maps to a list of audiences who may be able to get access to the collection.

The following audiences are defined:

* `public`: Open to the general public. If this is specified, any other values are redundant.
* `educational-primary`: Open to pre-university students.
* `educational-secondary`: Open to university-level students.
* `research`: Open to academics and researchers.
* `print-disability`: Open only to those who have a print disability.
* `other`: Open to people who meet some other qualification. This requirement should be explained in prose, in `service_description`.

Values are treated as inclusive. This value for `audiences` indicates that the collection is available to students of all ages.

```
"audiences": ["educational-primary", "educational-secondary"],
```

If no `audiences` are defined, a client may assume that an OPDS server is open to the general public.

A geographic restriction or a registration requirement does not qualify as an audience restriction in this sense. Those are handled separately, in `service_area` and with the `rel="register"` link. For example, a university library may have an `audience` of `["educational-secondary"]` and a `service_area` describing the city in which the university is located.

## `service_area`

Some libraries and bookstores only serve certain geographical areas. The `service_area` object lets an OPDS server specify its service areas. This helps clients guide people to the OPDS servers that serve their area.

The `service_area` object may take on different types of values:

* The literal string `everywhere`, indicating that the OPDS server considers the entire universe within its service area.
* A GeoJSON object that spells out the service area in geographic terms.
* A dictionary that maps [[ISO 3116-1 alpha-2 country codes|https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2]] to lists of place names.

A "list of place names" may take on two different values:

* A list of strings, each of which refers to a single place.
* The literal string `everywhere`, which means every relevant place.

If the `service_area` is not present, clients should assume a value of `everywhere`.

This spec does not define which strings refer to which places in a "list of place names". However, the Library Simplified library registry can understand the following place names for the United States.

* A state or territory, by its abbreviation.
* A census-designated place (such as a city or town) in the format "{city}, {state abbreviation}".
* A county in the format "{name} County, {state abbreviation}".
* A ZIP code, as a string.

Here's the `service_area` for a library that serves the entire state of California, one city in Kansas, one county in Florida, and one ZIP code in Illinois:

```
"service_area" : {
  "US": ["CA", "Lawrence, KS", "Broward County, FL", "60604"]
}
```

Here's a library that serves the United States plus one city in Canada:

```
"service_area" : {
  "US": "everywhere",
  "CA": ["Toronto, ON"]
}
```

Here's a library that serves everyone:

```
"service_area": "everywhere"
```

## `focus_area`

A library's `focus_area` is the geographic area it focuses on, within its `service_area`. For public libraries, this is sometimes called the library's jurisdiction.

For example, the New York Public Library has a `service_area` of New York State, but its `focus_area` is limited to three of the boroughs of New York City:

```
"service_area": { "US": ["NY"] },
"focus_area": { "US": ["Bronx County, NY", "New York County, NY", "Richmond County, NY"] }
```

The format for `focus_area` is the same as for `service_area`. If no value is specified for `focus_area`, the `focus_area` and `service_area` are assumed to be the same. The `focus_area` must be a geographic subset of the `service_area`, although this is not currently enforced.

Choosing a large `focus_area` can be bad for your search placement. Consider a university press in Kentucky that publishes an OPDS feed containing a few open-access titles. Since the content is all open-access, and anyone can download it, `everywhere` makes sense as a value for `service_area`. But specifying `everywhere` as a value for `focus_area` will put it in direct competition with much larger universal collections such as the Internet Archive. Specifying a more restrictive `focus_area` will boost the university press in search results for people in or near Kentucky.

```
"service_area": "everywhere",
"focus_area": {"US": ["KY"]}
```

## Link relation for anonymous access

The `https://librarysimplified.org/rel/auth/anonymous` link relation can be used as a value in the `type` array to indicate that it's okay to get books from this site without providing any authentication.

If `https://librarysimplified.org/rel/auth/anonymous` shares the `type` array with another authentication mechanism, it means that it's _also_ possible to authenticate with the site, possibly getting access to additional books or features.

# Standard features of special interest to SimplyE

## `"rel": "alternate", "type": "text/html"`

SimplyE expects an Authentication for OPDS document to link to the homepage of the organization that runs the server. For a public library, this would be the library's web site. This link has a `rel` of "alternate"` (an IANA-registered link relation) and a `type` of "text/html".

```
`"links" : [
 {
  "href": "http://www.nypl.org/",
  "rel": "alternate",
  "type": "text/html"
 }
]
```

## `"rel": "start"`

SimplyE expects an Authentication for OPDS document to link to the root feed of the OPDS server it describes, using the IANA-registered link relation "start".

## `"rel": "register"`

This link is used when someone doesn't currently have an account on the OPDS server, and wants to get one. If they follow the instructions at the other end of the URL, they should be in a position to close the web view and enter their newly created credentials. It won't work if there's an extra validation step where they have to, e.g. walk into a branch library and show ID.

Ideally the site at the other end of this URL would support the [Simple Signup Protocol](Simple-Signup-Protocol), but it's not required.

## `rel="logo"`

If the OPDS server is run by an organization that has an identifiable logo, linking to that logo from the Authentication for OPDS document, using `rel="logo"`, is a good way to help people find your server in a catalog.

For SimplyE we expect logos to be 135 by 135 pixels square, in PNG format, and to look good on a white background. We also prefer that logos be embedded in the Authentication for OPDS document using a [data: URL](http://dataurl.net/), rather than be external links that have to be fetched separately.

## `rel="support"`

TODO: We use rel="help" for some things where rel="support" would probably be better.

# Changes to the core spec

We propose some changes to the core OPDS For Authentication spec, mainly centered around these two issues:

1. Making the hypermedia link format consistent with that found in OPDS 2.0
2. Making it possible to unambiguously specify multiple different authentication flows of the same type.

## 2.3. Syntax
### 2.3.1. Core Properties

The Authentication Document MUST contain the following name/value pairs:

| Name |  Value | Format/data |
| ---- | ------ | ----------- |
| authentication | A list of Authentication Flows as defined in section 2.x. | Array of objects |

## 2.x. Authentication Flows

An Authentication Flow is a JSON object describing a specific method of authenticating with the OPDS Server.

An Authentication Flow must contain the following name/value pair:

| Name |  Value | Format/data |
| ---- | ------ | ----------- |
| type | Indicates the type of credentials being offered or the protocol used to authenticate a user. | URI |

An Authentication Flow object MAY include any or all of the following name/value pairs:

| name | value |
| ---- | ----- |
| `description` | As defined in 2.3.1. |
| `links`       | As defined in 2.3.2. |
| `labels`      | As defined in 2.3.3. |

### 2.3.2. Links

An Authentication Document or Authentication Flow MAY contain a
`links` object.  This is used to associate the Authentication Document
(or Authentication Flow) with resources that are not locally
available.

A `links` object is a list of link objects.

[Move name/value pairs for link objects above the list of link
relations, and add "rel" to the list of properties associated with a
link object.]

This specification defines the following `link` relations for the
links object:

| Relation | Semantics | Applies to | Required? |
| - | - | - | - |
| authenticate | Location where a client can authenticate the user with OAuth. | Authentication Flow only | Yes, if the Authentication Flow uses OAuth. |
| refresh | Location where a client can refresh the Access Token by sending a Refresh Token. |  Authentication Flow only | No |
| logo | Image of a logo associated with the Catalog Provider or Authentication Provider | Both Authentication Document and Authentication Flow | No |
| register | Location where a user can register with the Catalog Provider or Authentication Provider | Both Authentication Document and Authentication Flow | No |
| support | Support resources (a website, an email address, or a telephone number) for a user who is having problems with the Catalog Provider or Authentication Provider | Both Authentication Document and Authentication Flow | No |

A client SHOULd NOT mix the `links` associated with the main
Authentication Document with the `links` associated with an
Authentication Flow. The Authentication Document's `links` are
associated with the Catalog Provider; the Authentication Flow's
`links` are associated with the Authentication Provider. If a given
link applies to both the Catalog Provider and the Authentication
Provider, a server MUST specify it separately in both lists.

### 2.3.3. Labels

If an Authentication Flow does not define a value for `labels`, it
inherits the value for `labels` associated with the Authentication
Document itself. If an Authentication Flow defines a value for
`labels`, that value MUST be used instead of any value associated with
the Authentication Document.

If an Authentication Flow does not define a value for `description`,
it inherits the value for `description` associated with the
Authentication Document itself. If an Authentication Flow defines a
value for `description`, that value MUST be used instead of any value
associated with the Authentication Document.
