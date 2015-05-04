# Extra metadata

Wherever possible we use terms from Dublin Core or schema.org rather than making up our own terms.

## Availability information

```
   <simplified:total_licenses>0</simplified:total_licenses>
   <simplified:available_licenses>0</simplified:available_licenses>
   <simplified:active_holds>0</simplified:active_holds>
```

## Audience

```
<category term="Adult" scheme="http://schema.org/audience"/>
```

The terms we use are 'Adult', 'Adults Only', 'Young Adult', and 'Children'. It's kind
of a lie to have http://schema.org/audience as the scheme because
http://schema.org/audience doesn't actually control that vocabulary
(we made it up), it just says what an audience is.

## Target age

```
<category term="9" scheme="http://schema.org/typicalAgeRange"/>
<category term="9-12" scheme="http://schema.org/typicalAgeRange"/>
```

Only useful for children's and YA books. Currently only the single-number "9" format is supported.

## Publication date

```
<published>2014-04-01</published>
```

This is a standard part of Atom but I'd like to make it clear in the
OPDS spec that this we interpret this as the date this ebook was
commercially published (or otherwise released), not the date it was
accessioned into this collection, or the date of publication of the
print book, or the date of first publication, or the copyright date,
or the date the feed was generated. RFC 4287 is ambiguous: it says
published is "an instant in time associated with an event early in the
life cycle of the entry."

If I'm wrong about this, then I need to figure out if there's
a Dublin Core term that can mean "publication date for the ebook", or
create a new term.

It might be useful to specify how clients should interpret all the
different date terms, because IMO the Dublin Core terms are often
ambiguous when describing commercially published ebooks.

Maybe this doesn't matter in the larger context and I'm just too
uptight about defining exactly what dates mean, but for Gutenberg
texts the differences between these different dates can be hundreds of
years.

## Permanent work ID

```
<simplified:pwid>f819023c-a8a5-6d88-8c66-7a5c0c4ab249</simplified:pwid>
```

This is used for machine-to-machine integration. It's an opaque
identifier that identifies a work of literature independent of edition
or format. So Gutenberg's "Moby-Dick" and a commercial audiobook of
"Moby-Dick" and Feedbooks's "Moby-Dick" would all have the same pwid.
The algorithm for calculating pwid is developed by myself and Mark
Noble of the Colorado public library system.

This can't be <id> because an OPDS feed may contain different editions
of the same book.

I may also be adding a second pwid field (let's call it pwid-prime for
now) which identifies the work independent of edition, but which takes
format into account. So all print editions of 'Moby-Dick' would share
a pwid-prime, and all audiobooks of 'Moby-Dick' would share another
pwid-prime.

## Category weights

```
<category schema:ratingValue="2" term="sh98004865"
scheme="http://purl.org/dc/terms/LCSH" label="Paranormal fiction"/>
```

This is used in machine-to-machine integration so that the circulation
manager (which builds an index by topic) knows how heavily to weight a
given topic for a given book.

I think using schema:ratingValue here is a big hack.

## Detailed author information

```
<author>
     <name>F. Paul Wilson</name>
     <simplified:sort_name>Wilson, F. Paul</simplified:sort_name>
     <schema:family_name>Wilson</schema:family_name>
     <simplified:wikipedia_name>F._Paul_Wilson</simplified:wikipedia_name>
     <schema:sameas>http://viaf.org/viaf/100960913</schema:sameas>
</author>
```

Speaks for itself I think. Currently we only use this in
machine-to-machine integration, but you can see the value of being
able to link to an author's Wikipedia page.

## Estimated total size of partial feed

Something like this:

```
<link href="/Fiction/all"
      type="application/atom+xml;profile=opds-catalog;kind=acquisition"
      title="All fiction"
      total="70000"
/>
```

There's no implication that if you get the feed you will get all 70k
entries in a single document, just that that's the approximate total
size.

I plan to add this in the near future. I thought this was covered in
RFC 5005 but I couldn't find it. Maybe I was thinking of the thr:total
extension in RFC 4685.

It might be okay to put this information in the
title (e.g. "All fiction (70000)") but I don't like that because it
means the client can't make decisions about e.g. sorting collections by size or hiding collections that are miniscule.

## Medium of an entry

Something like this:

```
<entry schema:additionalType="http://schema.org/MusicRecording"/>
```

Simplified's circulation manager doesn't need this because we only show textual ebooks, but since there are non-textual items in our collection I need this during machine-to-machine integration. Otherwise the fact that a given item is a music recording tends to get lost.

# Metadata lookup protocol

This is a protocol for bulk machine-to-machine transfer of metadata. The metadata lookup protocol takes as its input one or more URNs that identify published books. It returns an OPDS feed with one entry for each URN. Each entry contains relevant metadata about that book, or a status message.

On a server that supports the lookup protocol the URI Template for the protocol is:

```
/lookup{?urn*}
```

Of course I can specify this URI template in a description document to get rid of the hard-coded template, but I haven't defined a description document format yet.

## URN format

For ISBNs, the URN format is the standard urn:isbn: format. For other ways of identifying books I've come up with a format based on {identifier type}/{identifier}. e.g.

```
urn:librarysimplified.org/terms/id/Gutenberg%20ID/100
urn:librarysimplified.org/terms/id/3M%20ID/aws4f
```

A given server will probably only be able to resolve a subset of these URNs. This is something that can be specified in the hypothetical description document, e.g. "I can tell you about a book if you give me an ISBN  or an Overdrive ID, but nothing else."

## Extensions to `<entry>`

The `<id>` of an entry in the OPDS feed returned by the lookup protocol is a URN provided by the client.

If the server has metadata to offer about the book identified by that URN, the entry contains that metadata.

Instead of metadata, the entry may contain a `status_code` and a `message` which explains why no metadata is forthcoming.

Some examples:

```
  <entry>
    <id>urn:librarysimplified.org/terms/id/Gutenberg ID/nosuchid</id>
    <simplified:status_code>400</simplified:status_code>
    <simplified:message>'nosuchid' is not a well-formed identifier</simplified:message>
  </entry>
```

```
  <entry>
    <id>urn:librarysimplified.org/terms/id/Gutenberg ID/1984</id>
    <simplified:status_code>404</simplified:status_code>
    <simplified:message>No text with identifier '1984'.</simplified:message>
  </entry>
```

```
  <entry>
    <id>urn:librarysimplified.org/terms/id/Gutenberg ID/100</id>
    <simplified:status_code>202</simplified:status_code>
    <simplified:message>You're the first one to ask about this identifier. I'll try to find out about it.</simplified:message>
  </entry>
```

The client should treat the `status_code` for a URN as if it had made an HTTP request to that URN and gotten that status code in response. In particular, if the client gets a `status_code` of 400 or 410 it should not ask about that URN again. A `status_code` of 404 does _not_ mean 'don't ask again'; it means that the controlling authority says the given identifier does not _currently_ identify any book.

## schema:sameAs

The lookup service may use schema:sameas to assert that, in its opinion, there are other URNs that identify the same work.

```
<link rel="http://schema.org/sameAs" href="urn:isbn:1208320490"/>
```

## Possible extensions

I'm hesitant to do this because I really don't want to recreate HTTP inside OPDS, but some sort of max-age data might be useful as a signal re: when it's okay to ask about this identifier again.

It's theoretically possible for an identifier to refer to more than one book. (e.g. ISBNs get reused). What happens then?

The URLs to this service can get incredibly long. For the sake of maximum compatibility with different web gateways I'd like to specify a way to do lookups over POST. This would probably involve POSTing a document of media type "text/uri-list" to the lookup URL.