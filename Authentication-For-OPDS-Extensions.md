Although any OPDS server can interact with any OPDS client, creating a seamless user experience requires the server to provide a lot of information not covered in the OPDS spec.

A simple example: OPDS doesn't say anything about authentication. Although library patrons are generally authenticated by a mechanism that's compatible with HTTP Basic Auth, that's not always true. Even when it is true, providing a generic HTTP Basic Auth that asks for "username" and "password" creates a bad experience for a library patron who doesn't think of their "library card number" or "barcode" as a "username".

The [Authentication for OPDS](https://docs.google.com/document/d/1-_0HHt664bDjybtCauBJXUSDXiT-Clg1sZUVNxHyLjw/edit#heading=h.r2fysm93j6kk) spec gives an OPDS server a way to explain how its clients should present the authentication interface. This document lists extensions the Library Simplified team has devised to give an OPDS server a way to explain _other_ things about the library that affect the user interface.

# Color scheme

An OPDS server may use the `color_scheme` extension to specify the color scheme a client should use when rendering that server's OPDS feeds.

`"color_scheme": "blue",`

The color schemes supported by SimplyE are "red", "blue", "gray", "gold", "green", "teal", and "purple". The specific colors used by these color schemes are laid out in the [NYPL Design Toolkit](https://nypl.github.io/design-toolkit/sections/color.html). ("Gold" is a nicer name for yellow.)

# Server description

An OPDS server may use the `service_description` extension to describe itself. This is distinct from the standard `description` field, which is to be used to describe the text prompt displayed to the authenticating user.

```
 "service_description": "Here you can get all sorts of free books!",
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

Libraries sometimes issue plastic cards that contain machine-readable representations of a patron's login identifier. The `barcode_format` extension lets an OPDS server explain how to acquire a value for a field through scanning an image rather than through keyboard input.

The following values are defined for this extension.

* `Codabar`: [[Codabar|https://en.wikipedia.org/wiki/Codabar]]

Unlike other field extensions, the `barcode_format` extension is only defined on the `login` field. It has no effect if defined on the `password` field.

# Feature flags

An OPDS server may wish to enable or disable certain common features of OPDS clients. Often this can be done by providing or omitting links with certain link relations, but when this isn't enough, a server can add a feature flag URI to the `feature_flags` extension.

Note that an OPDS client is not obligated to respect a feature flag--it can provide UI for a "disabled" feature and decide not to show UI for an "enabled" one.

The `features` extension object contains two optional keys, `enabled` and `disabled`. If present, each maps to a list of URIs indicating features that are enabled or disabled for that server.

The following URIs are defined for use as feature flags:

* `https://librarysimplified.org/rel/policy/reservations`: This feature is enabled by default. If it is disabled, a client should not show any indication that it's possible for a user to place a reservation for a title. A title is either available right now or it's not.

# Standard features of special interest to SimplyE

## `rel="help"`

## `rel="register"`

This link is used when someone doesn't currently have an account on the OPDS server, and wants to get one. If they follow the instructions at the other end of the URL, they should be in a position to close the web view and enter their newly created credentials. It won't work if there's an extra validation step where they have to, e.g. walk into a branch library and show ID.

Ideally the site at the other end of this URL would support the [Simple-Signup-Protocol|Simple Signup Protocol], but we can handle it if not.

## Logo
