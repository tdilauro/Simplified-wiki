Each library has some way of identifying and authenticating patrons. Authentication serves two purposes: to distinguish between patrons and non-patrons, and to identify resources associated with a particular patron.

Libraries most commonly authenticate patrons against an Integrated Library System (ILS). The Library Simplified circulation manager integrates with a few ILS systems through custom APIs, and with many more through the SIP2 protocol.

A library may support multiple authentication techniques. For example, the Open Ebooks collection authenticates patrons through both First Book and Clever. Currently, a circulation manager can be configured to support one HTTP Basic Auth mechanism and any number of OAuth mechanisms.

To connect your circulation manager to one or more authentication sources, define a `policies["authentication"]` section in your configuration file.

* `providers`: A list of authentication providers. At least one provider MUST be present.
* `bearer_token_signing_secret`: A secret used to sign OAuth bearer tokens before sending them to clients. If you define an OAuth provider, this MUST be present. If you define more than one OAuth provider, they will all sign keys using the same secret.
* `register_url`: If you provide this, unauthenticated patrons will be given the opportunity to sign up for an account rather than enter credentials. A patron who chooses to sign up for an account will be sent to the given URL. TODO: This is not implemented yet, and it's not clear how the client is supposed to know when to _close_ the web view.
