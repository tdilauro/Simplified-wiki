The Short Client Token is a short string that identifies the bearer as a particular patron of a particular library. A Short Client Token can be used to prove a patron's identity to a third party that does not know anything about how that library verifies its patrons.

The Short Client Token serves the same purpose as the JSON Web Token created by the [[Vendor ID Service|https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit]]. Unfortunately, these JWTs cannot be used to disassociate a device from an Adobe Account ID. Deactivating a device through Adobe requires a 'username' that must be at most 80 characters long, and a 'password' of at most 76 characters. In addition, certain characters cannot appear in the 'username' or 'password'. 

The Short Client Token is designed to provide the same basic security characteristics as a JWT from the Vendor Web Service, while being easily convertible into short 'username' and 'password' portions.

A Short Client Token contains no personally identifying information and cannot, on its own, identify the bearer as a particular human being.

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

# Signing the token

* Start with the string to be signed, e.g. `NYNYPL|1486651569|474f5ee0-a518-91e8-b71f-0e9c1d590815`. 
* Take the SHA-256 HMAC of the string.
* Encode the HMAC with the base64 algorithm.

Certain characters that are valid in base64 will be dropped or mangled by Adobe en route if you submit them as part of a password. After encoding the HMAC as base64, perform the following additional steps:

* Replace the plus character (+) with a colon character (:).
* Replace the slash character (/) with a semicolon character (;).
* Replace the equal sign (=) with an at sign (@).
* Strip newlines.

To construct the finished token, append a pipe character (|) onto the string to be signed and then append the encoded signature.

# Deriving 'username' and 'password'

Whenever a Short Client Token must be used to authenticate through a username/password mechanism such as Adobe device deactivation or HTTP Basic Auth, the token can be split into a 'username' component and a 'password' component.

The signature is the password; the part of the string that was signed is the username. The pipe character immediately preceding the password is not considered part of the username or the password.

To continue the example above, the 'username' would be `NYNYPL|1486651569|474f5ee0-a518-91e8-b71f-0e9c1d590815` and the 'password' would be `hap72czxMT98WjOgnWaLv1H4:wFKivwEk7qrfBJTN0Y@`.

