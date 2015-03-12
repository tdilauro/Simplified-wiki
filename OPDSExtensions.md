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

The terms we use are 'Adult', 'Young Adult', and 'Children'. It's kind
of a lie to have http://schema.org/audience as the scheme because
http://schema.org/audience doesn't actually control that vocabulary
(we made it up), it just says what an audience is.

I don't think we should try to create a complete controlled vocabulary
for Audience, because I may end up adding American school grades
("Grades 5-6") to the vocabulary. But agreeing on values for
Adult/Young Adult/Children might be a good idea.

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
<category term="http://schema.org/MusicRecording"
scheme="http://schema.org/additionalType"/>
```

Currently Simplified doesn't need anything like this because we only
show textual ebooks. I don't think it will suffice to
opds:indirectAcquisition, because sometimes I need to talk about a
book when there is no way of actually obtaining that book.