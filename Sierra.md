Sierra is an ILS that provides an API for patron authentication.

There are currently two active versions of the Sierra API: [Version
2](https://sandbox.iii.com/docs/v2/Content/zTutorials/tutAuthenticate.htm)
and [Version
3](https://sandbox.iii.com/docs/Content/zTutorials/tutAuthenticate.htm).

We currently believe that version 2 cannot be integrated with Library Simplified. Libraries that use version 2 of Sierra API should either upgrade to version 3 or install another API such as [Millenium].

## Setup

## Getting an authorization token with Client Credentials Grant

http://sandbox.iii.com/docs/Content/zReference/authClient.htm
http://sandbox.iii.com/docs/Content/zTutorials/tutAuthenticate.htm

## Getting a patron authorization token with Authorization Code Grant

http://sandbox.iii.com/docs/Content/zReference/authAuthCode.htm

## Getting patron information

Now we're authorized to act on behalf of a patron, but the only thing we really need to do is look up their identifying information.

https://sandbox.iii.com/iii/sierra-api/swagger/index.html#!/info/Get_token_information_get_0