You're a publisher or distributor. You have some books you've licensed
to one or more libraries. You want to [get these books into SimplyE](GettingYourBooksIntoSimplyE).

You want to host the books yourself, but make
them available to patrons of that library on demand. You don't want to
have to integrate with the library's ILS to check their patron
credentials; you trust the library to determine who should have access
to the books.

Here's the solution:

1. Put up an OPDS feed that lists the metadata for the books, and
   includes links to their download URLs. To actually download
   a book from its download URL requires a token.
2. Put up an OAuth token service that implements the [OAuth 2.0 Client
   Credentials](https://tools.ietf.org/html/rfc6749#section-1.3.4) flow.
3. Create an access key and secret key for the Client Credentials flow.
   Give these keys to the library who's licensed the books. (Create a
   different set of keys for every library.)
4. Put up an Authentication For OPDS document that links to your token
   service. Link to this document from your root OPDS feed.

Here's how it works:

1. When a patron of a library wants to download one of your books, they
   will contact a server operated by that library.
2. The library server will contact your server, using its access key
   and secret key, and ask for an OAuth bearer token good for 60 seconds.
3. When you return the OAuth bearer token, the library server will send
   this bearer token to the patron's device.
4. The patron's device will use the bearer token to download the book
   from your server.
5. After 60 seconds, the bearer token expires, preventing a patron from
   using it to download every book in your system.

Here's a more detailed look at the software you need to set up for
this system to work.

## OPDS feed

Your OPDS feed won't be shown directly to library patrons; it will be
consumed by a piece of software. It's important to include all the
metadata you would show to a patron (title, author, categories, and so
on), because that information will _eventually_ be shown to the
patrons, but there's no need to arrange the books in an order that humans will
find attractive. The library's server will be combining your books
with books from other sources, and presenting a unified feed (using
the metadata you provided) to its patrons.

Here are best practices for your machine-readable OPDS feed.

* List all the books, in reverse chronological order (newest
  first). If something about a book changes (its metadata, for
  instance), bump it to the top of the list.

* Don't put everything on one page; keep it to 50 or 100 books per
  page. Use the "next" link relation to link from one page to the
  next.

* Make sure each book has a link with the relation
  `http://opds-spec.org/acquisition`. This is where the patron will go
  to download the book. It should look something like this:

```
<entry>
<title>A Great Book</title>
...
<link href="https://my-opds-server/the-book.epub"
      rel="http://opds-spec.org/acquisition" type="application/epub+zip"/>
</entry>
```

# Client Credentials flow

Next, create a piece of software that implements the [OAuth 2.0 Client
Credentials](https://tools.ietf.org/html/rfc6749#section-1.3.4) flow.

Basically, you need to implement request-response interactions like
the following.

Request:

```
POST /get-a-bearer-token
Host: my-opds-server
Content-Type: application/x-www-form-urlencoded

client_id={client_id}&secret={client_secret}&grant_type=client_credentials
```

Response:

```
200 OK HTTP/1.1
Content-Type: application/json

{
    "expires_in": 60,
    "token_type": "Bearer",
    "access_token": "zKBkFyWYTYmrRGuER2SmpMc9y3qd8T"
}
```

The `client_id` and `client_secret` are the two pieces of information
you'll give to a library to allow them to access your books.

Your `expires_in` value should be at least 60 seconds. You need to
allow time for the library's server to propagate the access token to
the mobile client, and then for the mobile client to use it in a
request to your server.

You can issue a token that can only be used to download the specific
book, or a token that can be used to download any book on the
site. Whatever's easiest for you. The SimplyE mobile client will only
use the token it receives to download the specific book it requested.

A library patron with an `access_token` can use it to download a book
they would otherwise be unable to download:

```
GET /the-book.epub
Host: my-opds-server
Authorization: Bearer zKBkFyWYTYmrRGuER2SmpMc9y3qd8T
```

## Authentication For OPDS document

Now you have an OPDS server that allows an authenticated user to
download a book, and you have a service that allows a library to make
any patron an "authenticated user" for a limited time.

But there's no connection between these two pieces. There's no
indication that this is the kind of OPDS server that _also_ implements
the Client Credentials flow. Someone looking at the OPDS server can
try to get `/the-book.epub`, but they'll just get a 401
error. It's not clear what to do about it.

The Authentication For OPDS document connects these two pieces.

Your root OPDS feed should include a link to your Authentication For
OPDS document on the `<feed>` level (not inside any particular `<entry>`).

```
<feed>
<link href="https://my-opds-server/authentication-doc"
      rel="http://opds-spec.org/auth/document"
      type="application/vnd.opds.authentication.v1.0+json"/>
...
</feed>
```

You should also serve your Authentication For OPDS document along with
any 401 response code. Be sure to include its media type
(`application/vnd.opds.authentication.v1.0+json`) in the
`Content-Type` header.

Here's the simplest Authentication For OPDS document that will work in
this scenario:

```
{
  "id" : "https://my-opds-server/",
  "authentication": [
    {
      "type": "http://opds-spec.org/auth/oauth/client_credentials",
      "links" : [
        { "rel": "authenticate",
          "href" : "https://my-opds-server/get-a-bearer-token"
        }
      ]
    }
  ]
}
```

This document defines an `id`, which establishes which OPDS server
it's talking about.

This document also defines a single authentication flow, which
explains how to get past a 401 error. All you have to do is present an
OAuth Bearer Token that you got through the OAuth Client Credentials
flow. But which URL are you supposed to POST to, to trigger the Client
Credentials flow? That information is in the `authenticate` link.

So, if you try to download a book, but you get a 401 error, you're
served this document. You send a POST request to
`https://my-opds-server/get-a-bearer-token` and get a bearer
token. Then you repeat the request that got you the 401 error, but
this time you include the bearer token. Now you can download the book.

# Bearer Token Propagation

From an implementation perspective, that's all you need to know. But
you may have noticed some sleight-of-hand in the narrative just
above. The "you" who sends a POST request to
`https://my-opds-server/get-a-bearer-token` is different from the
"you" who downloads the book. The "you" who sends the POST is a
library; the "you" who downloads the book is one of the library's
patrons.

The library can get a bearer token with no problem, but how does it
communicate that bearer token to its patron?

You don't need to know how this works unless you are writing an OPDS
server for a _library_, but here's how it works:

## How It Works

There are three parties here: the patron (using a mobile device), the
library (a web server), and the distributor (another web server).

The patron is in the middle of an HTTP request. They have just asked
the library to send them a copy of a specific book. The library has
just obtained a bearer token `abcdefg` from the distributor. Anyone
who presents that token to the distributor can download that book.

The patron is waiting for an HTTP response. The library doesn't have
the book and send the book to the patron, but the library _can_ send a
document containing the two pieces of information necessary to
download the book from the distributor:

1. The book's URL is `https://my-opds-server/the-book.epub`. (Obtained from the distributor's OPDS feed.)
2. You can download the book by providing the bearer token `abcdefg`. (Obtained through the OAuth Client Credentials flow.)

Here's the response the library will send to the patron in this
case. The document below is based on the [access token
format](https://tools.ietf.org/html/rfc6749#section-4.1.4) defined by
OAuth 2.0. In fact, it's almost exactly the same as the access token
the distributor just sent to the library! The only differences are
that this document has a special media type (it's not
`application/json`), and it uses `scope` in a nontraditional way.

```
200 OK
Content-Type: application/vnd.librarysimplified.bearer-token

{
    "expires_in": 60,
    "token_type": "Bearer",
    "access_token": "zKBkFyWYTYmrRGuER2SmpMc9y3qd8T"
    "scope": "https://my-opds-server/the-book.epub"
}
```

These fields come directly from the distributor:

* `access_token`: The access token to use when downloading the book.
* `token_type`: This will always be the literal string `Bearer`.
* `expires_in`: The token is good for this number of seconds.

This field is set by the library to tell the client which book it
should be downloading:

* `scope`: The URL of the book for which the client requested a copy.

## Advertising Bearer Token Propagation

Since not every OPDS client can handle this authentication flow, the
library OPDS server needs to advertise it (using
`<opds:indirectAcquisition>` whenever it offers books from this
distributor. Here's what the link to borrow a book from this
distributor might look like:

```
<entry>
<title>A Great Book</title>
...
<link href="https://my-library.org/borrow/book1"
      rel="http://opds-spec.org/acquisition/borrow"
      type="application/vnd.librarysimplified.bearer-token">
  <opds:indirectAcquisition type="application/epub+zip">
</link>
</entry>
```

This is saying:

* You can borrow this book by making a request to
  https://my-library.org/borrow/book1.
* But you're not going to get back an EPUB!
* You're going to get a document of type `application/vnd.librarysimplified.bearer-token`...
* ...which you can _exchange_ for an EPUB.

If you're an OPDS client, and you can't handle a document of type
`application/vnd.librarysimplified.bearer-token`, then you shouldn't
give your user the impression they can get this book through you. They
can borrow the book, but you won't be able to go through the steps
necessary to download it.
