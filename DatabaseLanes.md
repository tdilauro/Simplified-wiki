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

A new sublane will have its parent lane's settings as a default, but
it should be easy to change this.

# User interface

This will suffice for nearly all libraries. (Open Ebooks is a notable exception, and we'll have to put custom lane definitions in the database to handle it.)

The goal here is to meet the needs of most libraries with a minimum of UI controls.

Automatically create a lane for each of the 178 (or whatever number genres) in each supported language.

In the user interface, display the tree structure of the lanes for a given language. Alongside each lane, display the number of works in the collection that would be available through the lane.

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

Display a list of custom lists and well-known data sources for custom lists (NYT, maybe Novelist). 

Admins may check a box next to a genre to show the lane for that genre, or uncheck a box to hide the lane for that genre. These lanes are never deleted, only hidden.

If you hide all of a lane's subgenres, then that lane is rendered as a big scrolling acquisition feed. If one or more of a lane's subgenres are visible, that lane is rendered as a grouped acquisition feed, with an "All {genre}" feed at the bottom that's rendered as a big scrolling acquisition feed.

An admin may drag-and-drop a custom list (or list source) beneath a lane. This creates a lane based on that custom list (or list source) which shows up as a sublane in the grouped acquisition feed for the parent lane.

# The `lanes` table

* `lane_id`: Unique ID

* `library_id`: Foreign key to `libraries`. Must be present.  Two
  libraries can have lanes that are identical, but one lane cannot be
  associated with more than one library. There are too many ways for
  library-specific information to slip in (custom list, 

* `parent_id`: Foreign key to `lanes`. Null indicates a top-level lane.

* `full_name`: A name unique to this library.

* `display_name`: The name to display to patrons. A library can have
  only one lane with `full_name` "Fiction", but it can have separate
  "Adult Fiction" and "YA Fiction" lanes, both with the `display_name`
  "Fiction". However, a lane cannot have two sublanes with the same
  display name. -- this will look confusing.

* `fiction`: This can take three values: "fiction", "nonfiction" or
  "both".  If this is set to "fiction" (or "nonfiction"), then only
  fiction (or nonfiction) will be included in book lists, and only
  fiction (or nonfiction) genres will be available in the genre list.

* `audiences`: List of audiences. If this is set, only books whose
  audience matches one in the list will be included. More than one
  audience may be selected.

* `target_age`: Only applicable for lanes with `audiences` that
  contain no material for adults. If this is set, only books suitable
  for the given target ages will be included.

* `genres` : Many-to-many relationship with `genres`. Only books classified
  under the given genres will show up.

* `excluded_genres` Many-to-many relationship with `genres`. Books
  classified under the given genres will not show up, even if they
  would have otherwise based on `genres`.

* `languages`: A list of languages. Only books in one of these
  languages will show up.

* `exclude_languages`: A list of languages. Books in all languages
  except these will show up.

  This is mutually exclusive with `languages`, so maybe they should
  be stored in the same database field. Or this should be removed 
  altogether.

* `media`: Either a medium or a list of media. For now this can always be
  "Book" but soon "Audiobook" will be a possibility

* `formats`: Either a format or a list of formats. Currently this will
  always be "Electronic" but "Codex" may eventually be a possibility.

* `license_source_id`: Only titles from this license source will be
  considered.

* `searchable`: If this is True, then the lane's feeds will issue an
  OpenSearch document that searches the lane. If not, the OpenSearch
  document will search the nearest _parent_ of this lane that is
  searchable. (That's the existing behavior. Rather than blindly
  reproduce it, this is a good opportunity to do user research and
  figure out what is helpful.)

* `list_data_source_id`: Only titles on a CustomList associated with this
  data source will be considered.

* `list_identifier_id` Only titles on this specific CustomList will be
  considered.

* `list_seen_in_previous_days`: For lanes that use CustomLists to
  build title must have been on a CustomList. Default is zero,
  indicating that the title must _currently_ be on the list.

* `list_ignores_parent`: In general, if you create a lane based on one
  or more CustomLists, books are only shown in that lane if they're on
  the list _and_ they would be shown in the parent lane. This lets you
  manage a single "Staff Picks" list instead of a "Fiction Staff
  Picks", "Nonfiction Staff Picks", "Spanish Staff Picks", etc.
    
  If you set this boolean, then books from the list will be shown even
  if they wouldn't be shown in the parent lane. This lets you show
  e.g.  nonfiction for people who like science fiction, selected
  'adult' books in the YA section, etc.
  
  We can add this feature later, since it's not something we support now.

* `root_for_patron_type`: A list of strings. Patrons whose external
  type is in that list will be sent to this lane when they ask for the
  root lane. (This is how we'll get 'root_lane' out of the Open Ebooks
  JSON configuration.)

* `visible`: True by default. If this is set to false, the lane will
  not actually be shown to patrons. This lets librarians work on a
  lane without making it public.

* `appeals`: A list of appeals. We don't use appeals anymore so this can
  be removed for now.

# The `lane_genres` join table

This join table says that books in a given genre are eligible for
inclusion in a lane (or explicitly excluded from the lane).

* `lane_id`
* `genre_id`
* `include` - If this is True (the default) then this row means to
  include books that match the criteria. If this is False, then this row
  means to exclude books that match the criteria, even if they would
  otherwise be included.
* `with_subgenres` - If this is True, then all of the genre's
  subgenres are counted as this genre for purposes of inclusion (or
  exclusion) in this lane. If this is False, then the genre's 

These items are removed: If you want or don't want sublanes, you can
add them yourself. However, we do need to keep around the logic behind
these, for when we are initially populating the `lanes` table.

* `include_best_sellers`: 
* `include_staff_picks`
* `include_all`
* `subgenre_behavior`

# Initial state

To start with, there are no lanes.

A set of lanes can be autogenerated to match the current collection.

A lane must have 1000 books in it; otherwise it is folded up into the
next highest lane.

A key indicator of completeness is 'how many books are not in any lane?'

