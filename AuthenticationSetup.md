Each library has some way of identifying and authenticating patrons. Authentication serves two purposes: to distinguish between patrons and non-patrons, and to identify resources associated with a particular patron.

Libraries most commonly authenticate patrons against an Integrated Library System (ILS). The Library Simplified circulation manager integrates with a few ILS systems through custom APIs, and with many more through the SIP2 protocol.

A library may support multiple authentication techniques. For example, the Open Ebooks collection authenticates patrons through both First Book and Clever. Currently, a circulation manager can be configured to support one HTTP Basic Auth mechanism and any number of OAuth mechanisms.

To connect your circulation manager to one or more authentication sources, define a `policies["authentication"]` section in your configuration file.

# Quick start: Mock authentication

You can get a circulation manager set up without connecting to an ILS at all.

```
"authentication": {
    "providers": [
        { "module": "api.mock_authentication",
          "patrons": { "patron1": "password",
                       "patron2": "password2" }
        }
    ]
}
```

Here's what the configuration options mean:

* `module`: (REQUIRED) Indicates that authentication happens through the mock authenticator (as opposed to some other method).
* `patrons`: (REQUIRED) A dictionary mapping patron identifiers to passwords. You'll be able to authenticate through HTTP Basic Auth with an identifier/password combination found in this list. Any other identifier/password combination will fail authentication.
* `expired_patrons`: (OPTIONAL) A dictionary mapping patron identifiers to passwords. Patrons in this list will be able to authenticate, but will not be able to borrow books, because their credentials have expired.
* `patrons_with_fines`: (OPTIONAL) A dictionary mapping patron identifiers to passwords. Patrons in this list will be able to authenticate, but will not be able to borrow books because of their excessive fines.

You should be able to borrow books from Bibliotheca or Axis 360 using mock authentication. You will _not_ be able to borrow books from Overdrive using mock authentication, because Overdrive double-checks with the library's actual ILS before issuing a loan. 

# The `policies['authentication']` section in detail

The `policies['authentication']` section takes the following configuration options.

* `providers`: A list of authentication providers. At least one provider MUST be present.
* `bearer_token_signing_secret`: A secret used to sign OAuth bearer tokens before sending them to clients. If you define an OAuth provider, this MUST be present. If you define more than one OAuth provider, they will all sign keys using the same secret.
* `register_url`: If you provide this, unauthenticated patrons will be given the opportunity to sign up for an account rather than enter credentials. A patron who chooses to sign up for an account will be sent to the given URL. TODO: This is not implemented yet, and there are several undecided issues surrounding it.

In addition, each authentication source needs its own configuration.

# Which authentication mechanism do I use?

Sometimes just knowing which authentication mechanism you should use is difficult. The Sierra ILS offers two APIs that can be used with a Library Simplified circulation manager, plus three _different_ APIs with _very similar names_ that _sound_ like they can be used with a Library Simplified circulation manager, but actually can't.

For most American libraries, the simplest thing to do is to find an answer to this question:

_How does Overdrive communicate with my ILS?_

Most libraries have an Overdrive account.

