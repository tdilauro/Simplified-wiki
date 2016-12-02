Sierra provides a number of APIs for patron authentication. The one they're pushing heavily is the REST API, as opposed to the [[Millenium]] and SOAP APIs, which they consider to be "legacy".

There are currently two active versions of the Sierra REST API: [Version
2](https://sandbox.iii.com/docs/v2/Content/zTutorials/tutAuthenticate.htm)
and [Version
3](https://sandbox.iii.com/docs/Content/zTutorials/tutAuthenticate.htm). Version 1 may still be in use in some libraries.

It should be possible to use [[SIP]] to integrate any version of Sierra with a Library Simplified circulation manager. However, with version 3 of the Sierra API it also becomes possible to integrate Sierra with a Library Simplified circulation manager through an OAuth Authorization Code Grant.

We believe that version 2 cannot be integrated with Library Simplified except through SIP. Libraries that use version 2 of Sierra API should either connect via SIP, upgrade to version 3 or install another API such as [[Millenium]]. 

Furthermore we believe that even with version 3, it is not possible to authenticate a patron through the Sierra REST API and then borrow a book for that patron through Overdrive.

It may be possible to integrate with an ILS indirectly thorough a library's Overdrive account, but this is not a good long-term solution because it creates a major dependency on Overdrive.

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

You can use the access token as a `Bearer` token to make API requests that don't require acting on behalf of a particular patron.

Since the circulation manager only needs to interact with the ILS when acting on behalf of a particular patron, this will probably never be used in production code, but it might be useful when troubleshooting problems with the ILS.

## Getting a patron authorization token with Authorization Code Grant

This is a standard OAuth Authorization Code Grant, documented [here](http://sandbox.iii.com/docs/Content/zReference/authAuthCode.htm).

Let's say a patron wants to check their loans. They're going to get a 401 error. We don't know which patron they are. We will send them an OAuth authentication document with the 401 error and that document will tell them to log in to the ILS at a URL like `https://{root}/authorize?client_id={client ID}&redirect_uri={application URI}&state={state}&response_type=code`.

Example: `https://sandbox.iii.com/iii/sierra-api/authorize?client_id=sierra-client-id&redirect_uri=https://my-circulation-manager.com/oauth_callback%3Fprovider=Sierra&state=some-state&response_type=code`

`state` can be set to any value. `redirect_uri` is the URI defined in the initial setup step, the link into the circulation manager's `oauth_callback` controller.

The client will open a web view to the `iii.com` URL [QUESTION: will this work given the common practice of hiding this server behind an IP whitelist?]. That URL will redirect the patron to a Sierra login form. They will log in and will be sent to a URL based on the `redirect_uri`, something like: `https://my-circulation-manager.com/oauth_callback?provider=Sierra&code=3ui+a98th1i4&state=some-state`. In my experience there is no separate step asking them to authorize the Simplified app.

The circulation manager must not continue if the `state` that comes in here does not match the `state` that was sent out in the OPDS authorization document.

Assuming the `state` does match, the circulation manager then needs to exchange the `code` for an access token.

This works the same way as the Client Credentials Grant described above. You put your client key and secret into a Basic Auth request. The only difference is the entity-body:

```
grant_type=authorization_code&code=3ui+a98th1i4
&redirect_uri=redirect_uri=https%3A%2F%2Fmy-circulation-manager.com%2Foauth_callback%3Fprovider%3DSierra'
```

The response will include an access token good for one hour. We can use this token as a `Bearer` token to act on behalf of a patron. More importantly, we now know that the user of the Simplified app is a real, authorized library patron.

## Getting patron information

Now we're authorized to act on behalf of a patron, but the only thing we really need to do is look up their identifying information. The first step is to get their patron ID. This is documented as the ["Get token infromation"](
https://sandbox.iii.com/iii/sierra-api/swagger/index.html#!/info/Get_token_information_get_0) endpoint. That tells you which patron we're looking at. You can then use ["Get a patron by record ID"](https://sandbox.iii.com/iii/sierra-api/swagger/index.html#!/patrons/Get_a_patron_by_record_ID_get_9) to get detailed information about the patron, for purposes such as determining the patron type and outstanding fines.

This endpoint is not present in version 2 of the Sierra API. ([Example URL](https://lci-tr.iii.com/iii/sierra-api/swagger/index.html), will not be available outside NYPL.) This is why we say that version 2 can't be integrated into Library Simplified.

## Reauthorizing

Once the hour is up, the circ manager is going to have to start sending 401s again, and the client is going to have to open up that web view again. If the client's session cookie is still around in the web view, the patron will still be logged in and the token refresh can be done in the background, without patron intervention. The client's session cookie is a standard Tomcat JSESSIONID cookie that expires at the close of the sesssion. So long as the "session" stays open on the mobile device and the Sierra instance doesn't get restarted, the client will be logged in indefinitely.

## Logging out

The flip side of this is there seems to be no way to tell Sierra to log a patron out. A patron who logs out of the Simplified app will be immediately reauthorized (as per above) on the next 401 error. However we can force a logout on the client side by clearing the web view's cookies. The next time they get a 401 error they'll be asked to log in again.

## Some sample code

```
class SierraAPI(object):

    def __init__(self, host, key, secret,
                 redirect_uri='http://localhost:6500/oauth_callback'):
        self.host = host
        self.key = key
        self.secret = secret
        self.redirect_uri = redirect_uri

    def login_url(self, state):
        qs = urllib.urlencode(dict(state=state, response_type='code',
                                   redirect_uri=self.redirect_uri,
                                   client_id=self.key)
        )
        return self.host + '/iii/sierra-api/authorize?%s' % qs
        
    def get_token(self):
        url = self.host + '/iii/sierra-api/v2/token'
        basic = base64.encodestring("%s:%s" % (self.key, self.secret)).strip()
        headers = {'Authorization': "Basic %s" % basic}
        response = requests.post(url, headers=headers)
        data = json.loads(response.content)
        return data['access_token']

    def patron_lookup(self, bearer, barcode):
        url = self.host + '/iii/sierra-api/v2/patrons/find?barcode=%s' % barcode
        headers = {'Authorization': "Bearer %s" % bearer}
        response = requests.get(url, headers=headers)
        return response.content
```