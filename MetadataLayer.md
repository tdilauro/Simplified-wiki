Every API has a different way of representing bibliographic metadata
and licensing data. The metadata layer abstracts away the differences
between APIs, and hides the complexity of the underlying data model.

When you learn something about a title in the collection, you need to
find out as much bibliographic information about that title as
possible, and create an `Identifier`, `Edition`, `LicensePool`, and
`Work` for it. The simplest way to do this is through the metadata
layer.

The metadata layer is defined in `core/metadata_layer.py`. 

## `CirculationData`

A `CirculationData` object contains information about a license for a
book. This includes how many copies of the title are available and in
which formats. You can provide all the necessary items in the
constructor:

* `data_source`: The name of the data source that provides the
  license. For your integration this will always be the constant you
  added to the `DataSource` class.

* `primary_identifier`: An `IdentifierData` object that lets the
  circulation manager know how it should refer to the title when
  talking to the license provider. This is usually some sort of
  proprietary ID, but sometimes it's ISBN.

* `licenses_owned`: The number of licenses owned by the library that
  licensed this collection. If this collection doesn't track
  single-use licenses, it's okay to set this to 1 or 0, depending on
  whether the title is currently in the collection.

* `licenses_available`: The number of licenses that can be loaned out
  _right now_. If this collection doesn't track single-use licenses,
  it's okay to set this to 1 or 0, depending on whether the title can
  be loaned out right now.

* `patrons_in_hold_queue`: The number of patrons who are waiting in
  line to borrow this book. If this collection doesn't track
  single-use licenses, this will probably always be 0.

* `licenses_reserved`: The number of licenses that are in the
  following (somewhat specialized) state: someone put the book on hold
  waited for it, and is now at the front of the queue. The book is
  available _for them_ to borrow, but no one else can borrow it.
  
  If you don't know how many titles are in this state, it's okay to
  set this to 0.

* `formats`: A list of `FormatData` objects that explain what formats
  have been licensed. There must be at least one `FormatData` object,
  or there will be no way to deliver the title to a patron.

There are two other constructor arguments designed for use with
open-access collections:

* `default_rights_uri`: The terms under which this title is made
  available to the general public (_not_ to the library that licensed
  this collection) This should be one of the constants defined in
  `core/model.py:RightsStatus`. The default is `IN_COPYRIGHT`, which
  is appropriate for any title that is not open-access.

* `links`: This list may contain one or more `LinkData` objects which
  link to actual copies of the book.

## `Metadata`

A `Metadata` object contains bibliographic information about an item.

* `data_source`: The name of the data source that provides the
  metadata. For your integration this will always be the constant you
  added to the `DataSource` class.

* `title`: The title of the item.
* `subtitle`: The item's subtitle, if any.
* `sort_title`: The item's sort title, if different from the `title`.
* `language`: The primary language of the item. This should be a
  three-character ISO 639-2 code like "eng".
* `medium`: The medium of the item. This is one of the `_MEDIUM`
  constants from `core/model.py:Edition`; usually `BOOK_MEDIUM` or
  `AUDIO_MEDIUM`.
* `series`: If this item belongs to a series of items, the name of the series.
* `series_position`: The item's numeric position within its series, if it has
  one.
* `publisher`: The publisher that issued this electronic edition.
* `imprint`: The imprint of `publisher`, if any, that issued this electronic
  edition.
* `issued`: A `datetime object indicating when this electronic edition
  of the work was first made available. This date should be within
  the past few years.
* `published`: A `datetime` object indicating when this _work_ was
  originally published. This date might be hundreds of years in the past.
* `primary_identifier`: An `IdentifierData` object that lets the
  circulation manager know how it should refer to the title when
  talking to the data provider. This is usually some sort of
  proprietary ID, but sometimes it's ISBN.
* `identifiers`: A list of `IdentifierData` objects representing other
  ways the data provider has of referring to this title. There
  really ought to be an ISBN in here.

* `subjects`: A list of `SubjectData` objects representing the
  subjects under which the data provider thinks this title should be filed.

* `contributors`: A list of `ContributorData` objects representing the
  authors and other people who contributed to this title.

* `measurements`: A list of `MeasurementData` objects representing
  measurements the data provider has taken of this title.

* `links`: A list of `LinkData` objects representing related resources
  such as cover images.

* `recommendations`: A list of `IdentifierData` objects representing
  _other_ titles that the data provider recommends for people who liked
  this title.


* `circulation`: An optional `CirculationData`. If your API provides
  metadata and circulation information at the same time, you can
  create a `CirculationData` object and carry it around inside a
  `Metadata` object.

## `ContributorData`

TBD

## `SubjectData`

TBD

## `IdentifierData`

TBD

## `LinkData`

TBD

## `MeasurementData`

TBD

## `FormatData`

TBD

## `ReplacementPolicy`

When you call an `apply` method, you're supposed to pass in a
`ReplacementPolicy` object as well.

TBD
