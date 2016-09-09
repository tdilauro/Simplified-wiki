Sierra is an ILS that provides an API for patron authentication.

There are currently two active versions of the Sierra API: [Version
2](https://sandbox.iii.com/docs/v2/Content/zTutorials/tutAuthenticate.htm)
and [Version
3](https://sandbox.iii.com/docs/Content/zTutorials/tutAuthenticate.htm). Version 1 may still be in use in some libraries.

We currently believe that version 2 cannot be integrated with Library Simplified. Libraries that use version 2 of Sierra API should either upgrade to version 3 or install another API such as [[Millenium]]. It may also be possible to integrate with an ILS indirectly thorough a library's Overdrive account, but this is not a good long-term solution because it creates a major dependency on Overdrive.

## Setup

To set up Sierra for integration with your Library Simplified circulation manager, you'll need to create an API key and a secret. This information will go into the circulation manager configuration. You'll also need to associate a redirect URL with your API key. The redirect URL should point to the `oauth_callback` controller in your circulation manager:

```
https://my-circulation-manager.com/oauth_callback?provider=Sierra
```

## Root URL

Sierra systems seem to all be hosted at the `iii.com` domain, e.g. https://my-library.iii.com/. The root of the Sierra API seems to be `https://my-library.iii.com/iii/sierra-api/v2` (for version 2) or `https://my-library.iii.com/iii/sierra-api/v3` (for version 3).

## Getting an access token with Client Credentials Grant

If you don't need to act on behalf of a particular patron, you can use your client key and secret to get an access token. Encode the client key and secret into a Basic Auth request to the `token` endpoint, e.g. `https://my-library.iii.com/iii/sierra-api/v3/token` or `https://my-library.iii.com/iii/sierra-api/v2/token`.

The entity-body should be the string `grant_type=client_credentials`.

This is covered [here](http://sandbox.iii.com/docs/Content/zReference/authClient.htm).

Since the circulation manager only needs to interact with the ILS when acting on behalf of a particular patron, this will probably never be used in production code, but it might be useful when troubleshooting problems with the ILS.

## Getting a patron authorization token with Authorization Code Grant

This is a standard OAuth Authorization Code Grant, documented [here](http://sandbox.iii.com/docs/Content/zReference/authAuthCode.htm).

To act on behalf of a specific patron we need to get that patron to log in to the ILS. This starts by opening a web view to the URL `https://{root}/authorize?client_id={client ID}&redirect_uri={application URI}&state={state}&response_type=code`.

Example: `https://sandbox.iii.com/iii/sierra-api/authorize?client_id=sierra-client-id&redirect_uri=https://my-circulation-manager.com/oauth_callback%3Fprovider=Sierra&state=some-state&response_type=code`

`state` can be set to any value. `redirect_uri` is the URI defined in the initial setup step, the link into the circulation manager's `oauth_callback` controller.

The patron will be redirected to a Sierra login form. They log in, authorize the Simplified app, and are sent to a URL based on the `redirect_uri`, something like: `https://my-circulation-manager.com/oauth_callback?provider=Sierra&code=3ui+a98th1i4&state=some-state`.

The circulation



## Getting patron information

Now we're authorized to act on behalf of a patron, but the only thing we really need to do is look up their identifying information.

https://sandbox.iii.com/iii/sierra-api/swagger/index.html#!/info/Get_token_information_get_0