Sometimes you expect to see a certain book in your circulation manager's OPDS feed, but it just doesn't show up. There are a number of things that might have gone wrong.

# What to Do

The simplest thing to do is to run the `bin/repair/where_are_my_books` script. This will force-refresh various parts of the system that are normally refreshed on a slow timescale -- once an hour or once a day. On a new system, where _none_ of your books are showing up in feeds, `where_are_my_books` will also diagnose systemic problems that might indicate a problem with your importer.

If `where_are_my_books` doesn't fix your problem, you need to pick a specific book and see what's wrong with it. The best entry point is the `bin/informational/explain` script. You `explain` a book by passing in its primary identifier:

`$ bin/informational/explain --identifier-type="Axis 360 ID" 0010163843`

`$ bin/informational/explain --identifier-type="Overdrive ID" 019f21e3-9de9-4c40-95a4-dfabf55e7801`

`$ bin/informational/explain --identifier-type="URI" http://www.gutenberg.org/ebooks/289`

Or you can run it with the internal database ID of the `Identifier`:

`$ bin/informational/explain --identifier-type="Database ID" 101`

The output looks like this:

```
Assassin's Apprentice (J. B. Redmond, S. R. Vaught, Book) according to Axis 360
 Permanent work ID: c86ae1a7-1746-256c-9b3a-9c1e161cb99c
 Metadata URL: http://metadata.alpha.librarysimplified.org/lookup?urn=urn:librarysimplified.org/terms/id/Axis%20360%20ID/0010163843
 Primary identifier: Axis 360 ID/0010163843 (q=1)
   Identifier: ISBN/9781599908014 (q=1.0)
 Contributor[38]: contributor_sort_name=Redmond, J. B., contributor_display_name=None, 
 Contributor[39]: contributor_sort_name=Vaught, S. R., contributor_display_name=None, 
Licensepool info:
 Delivery mechanisms:
  Fulfillable application/epub+zip/application/vnd.adobe.adept+xml
 10000 owned, 9996 available, 0 holds, 0 reserves
Work info:
 Identifier of presentation edition: Axis 360 ID/0010163843 ID=101 prim_ed=51 ("Assassin's Apprentice")
 Fiction: True
 Audience: Young Adult
 Target age: NumericRange(15, 19, '[)')
 0 genres.
 License pools:
  ACTIVE: Axis 360 ID/0010163843 ID=101 prim_ed=51 ("Assassin's Apprentice")
```

This should help you identify which problem applies to your book. (The known possible problems are listed immediately below.)

# What Might Go Wrong

* The book must have been imported correctly. It must have `Identifier`, an `Edition`, a `LicensePool`, and it must belong to a `Work`. Getting this right depends on the `Monitor` that imported the book in the first place.
* The `Work` must be presentation-ready: it must have the minimal bibliographic information necessary to show to library patrons, and it must have gone through the classification process. Again, this is the responsibility of the importing `Monitor`.
* The `Work` must have a cached OPDS entry. This is created as part of the task of making the `Work` presentation-ready, but it can be redone by running the `bin/repair/work_opds` script.
* The `LicensePool` must be associated with at least one compatible `DeliveryMechanism`. If the book is not available in any particular format, or it's only available in formats that are not compatible with SimplyE, it will not show up in feeds. This, too, is the responsibility of the importing `Monitor`.
* `LicensePool.owned_licenses` must be at least 1. If the library does not own any licenses for the book, it won't show up. If holds are disabled for the library, then `LicensePool.available_licenses` must also be at least 1.
* The `LicensePool` must not be suppressed. Suppressing a `LicensePool` is a largely manual process done through the administrative interface.
* The `Work` must be present in the materialized views, which are used to build the lanes. The materialized views are updated once a day, through the `bin/refresh_materialized_views` script.
* The `CachedFeeds` cache may contain an old version of a lane which was built before the book was imported, or before some other problem with the book was fixed. Since the materialized view is not being queried, this can make it look like a book is still not in the collection. By default, list-type feeds are cached for twenty minutes. Grouped feeds are cached forever, until they are replaced by running `bin/cache_opds_blocks`.
* If a book shows up in list-type feeds but not in search results, it may not have been added to the ElasticSearch index. This can be repaired using the `bin/repair/search_index` script.
