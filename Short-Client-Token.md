The Short Client Token is a short string that identifies the bearer as a particular patron of a particular library. A Short Client Token can be used to prove a patron's identity to a third party that does not know anything about how that library verifies its patrons.

The Short Client Token serves the same purpose as the JSON Web Token created by the TK. Unfortunately, these JWTs cannot be used to disassociate a device from an Adobe Account ID. Deactivating a device through Adobe requires a 'username' that must be less than 75 characters and a 'password' of less than 80 characters. In addition, certain characters cannot appear in the 'username' or 'password'. 

The Short Client Token is designed to provide the same basic security characteristics as TK, while being easily convertible into a 'username' portion and a 'password' portion.

A Short Client Token contains no personally identifyin information and cannot, on its own, identify the bearer as a particular human being.

# Format

A Short Client Token has four parts, separated by the pipe character. Here's an example:

```
NYNYPL|1486651569|474f5ee0-a518-91e8-b71f-0e9c1d590815|hap72czxMT98WjOgnWaLv1H4:wFKivwEk7qrfBJTN0Y@
```

The parts are as follows:

1. A short string that uniquely identifies a library.
2. The expiration time of the token.
3. A string that uniquely identifies a patron.
4. A signature.

# Library name

The library name must be unique among all libraries in a federation. It should not exceed ten characters. For US public libraries in the SimplyE federation, we've adopted a convention where the first two letters of the library name designate the US state covered by the library.

# Expiration time

The expiration time is measured in integer number of seconds since midnight UTC on January 1, 1970. An otherwise valid token that has expired is considered invalid.

# Patron identifier

Since the patron identifier is stored in a database not under your control, this value MUST NOT be any value that might change over time or which contains personally identifying information. Unacceptable values include the patron's barcode, account number, email address, or username. Recommend best practice is to create a brand new identifier used solely to identify a given patron in Short Client Tokens. In the example above, a patron is identified by a UUID.

# Signature

# Deriving 'username' and 'password'

Whenever a Short Client Token must be used to authenticate through a username/password mechanism such as Adobe device deactivation or HTTP Basic Auth, the token can be split into a 'username' component and a 'password' component.

The signature is the password; everything else is the username. The pipe character immediately preceding the password is not considered part of the username or the password.

To continue the example above, the 'username' would be `NYNYPL|1486651569|474f5ee0-a518-91e8-b71f-0e9c1d590815` and the 'password' would be `hap72czxMT98WjOgnWaLv1H4:wFKivwEk7qrfBJTN0Y@`.

