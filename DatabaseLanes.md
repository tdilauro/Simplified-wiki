Types of lanes:

* Based on a language or languages
* Based on a format (audiobooks vs ebooks)
* Based on a license data source
  (e.g. put the entire Plympton collection up front)
* Based on an audience (adult vs YA vs children)
* Based on fiction status (fiction vs nonfiction)
* Based on genres
* Based on a custom list (Staff Picks)
* Based on a custom list data source (NYT Best Sellers)

These are general guides and lanes may combine different aspects. Most of a library's lanes will
specify a language, a format, and some other thing (usually a genre).

The feature branch tracking this work is [`database-lanes`](https://github.com/NYPL-Simplified/server_core/pull/683).

# User interface

This will suffice for nearly all libraries. (Open Ebooks is a notable exception, and we'll have to put custom lane definitions in the database to handle it.)

The goal here is to meet the needs of most libraries with a minimum of UI controls.

Automatically create a lane for each of the 178 (or whatever number genres) in each supported language.

In the admin interface, display the tree structure of the lanes for a given language. Alongside each lane, display the number of works in the collection that would be available through the lane.

```
+ English (1000)
  +- Fiction (500)
     +- Romance (50)
        +- Historical Romance (10)
        +- Paranormal Romance (0)
  +- Nonfiction
  +- Audiobooks
  +- YA
  +- Childrens
```

First, an admin may show or hide any lane in the tree from the patron interface. These lanes are never deleted, only hidden.

If you hide all of a lane's sublanes, then that lane is rendered as a big scrolling acquisition feed. If one or more of a lane's sublanes are visible, that lane is rendered as a grouped acquisition feed, with an "All {lane name}" feed at the bottom that's rendered as a big scrolling acquisition feed.

In addition, an admin may create a new sublane under one of the existing lanes, based on a custom list or a well-known data source for custom lists (NYT, maybe Novelist). The admin interface has a separate page for creating new lists. The lane editor should show the existing lists and link to the page for creating a new list.

The admin may remove a lane that was created from a list, and change the name of a lane that was created from a list. (This is different than the other non-list lanes, which can be hidden but not edited or removed). The admin may also specify whether a list-based lane should inherit restrictions from the parent lane. For example, if you had a custom list of non-fiction books about Science Fiction and wanted to put that under the Science Fiction lane, the lane should not inherit the parent's restrictions because that would restrict it to fiction books. On the other hand, if you had a general list of best-sellers and wanted to make a "Best Sellers" lane under Romance, inheriting the parent lane's restrictions would filter out the non-romance books.

An admin may also create a lane from multiple lists, or add an additional list to an existing list-based lane, but this isn't critical and doesn't need to be in the first version.

Finally, an admin may manage languages to control how lanes are automatically created. Each supported language may be treated as 'large', 'small', or 'tiny'. Large languages will have multiple separate lanes at the top level. Small languages will have one lane at the top level, and tiny languages will be grouped into 'Other languages' at the top level. Changing the setting for a language will recreate the lanes for that language and wipe out any list-based lanes that have been created under it.

# Initial state

When a library is initially created, it has no lanes. In the absence of a lane editor, a script runs once a week and establishes the following situation:

* For a language with a large collection, lanes are created a la `lanes_for_large_collection`. Any genre-specific lanes that contain fewer than 200 books are marked as hidden.

* For a language with a small collection, lanes are created a la `lane_for_small_collection`.

* If there are any other languages present, an 'other languages' lane is created with a lane for each such language, a la `lane_for_other_languages`.

When you create a library, you can run this script manually after importing a large number of books to get your lanes to show up.