Every API has a different way of representing bibliographic metadata (facts about a book, like its title)
and licensing data (facts about access to a book, like the number of patrons in its holds queue). The metadata layer abstracts away the differences
between APIs, and hides the complexity of the underlying data model.

When you learn something about a title in the collection, you need to
find out as much bibliographic information about that title as
possible, and create an `Identifier`, `Edition`, `LicensePool`, and
`Work` for it. The simplest way to do this is through the metadata
layer.

The metadata layer is defined in [`core/metadata_layer.py`](https://github.com/NYPL-Simplified/server_core/blob/master/metadata_layer.py). 

# `CirculationData`

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
  it's okay to treat this as a boolean: set it to 1 or 0, depending on whether the title can
  be loaned out right now.

* `patrons_in_hold_queue`: The number of patrons who are waiting in
  line to borrow this book. If this collection doesn't track
  single-use licenses, this will probably always be 0.

* `licenses_reserved`: The number of licenses that are in the
  following (somewhat specialized) state: someone put the book on hold,
  waited for it, and is now at the front of the queue. _They_ can
  borrow the book right now, but it's not available to borrow in general.
  
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

Note that there is no special place to put a description of the book, as you might find on the back cover. A book's description is contained in a `LinkData` object (see below) with `rel="description"`, which goes into `links`.

## `ContributorData`

The `Metadata` class acts as a container for lots of other objects. Like `Metadata`, these objects are convenient stand-ins for data model objects. Unlike `Metadata`, you don't need to call `apply()` on these objects to write them to the database. You just associate them with a `Metadata` and they're written to the database with everything else when you call `Metadata.apply()`.

The most important of these objects is `ContributorData`, which represents a person or person-like entity that contributed to the production of a title. It can contain the following information:

* `display_name`: The person's name as it would show up on the front of a book jacket, e.g. "Stephen King".
* `sort_name`: The person's name as it would show up in an alphabetized card catalog, e.g. "King, Stephen".
* `family_name`: The person's family name, as it might show up on the side of a book jacket, e.g. "King".
* `roles`: A list of roles this person had in the creation of this book. The `Contributor` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) lists about thirty common roles ranging from `EDITOR_ROLE` to `LYRICIST_ROLE`. The ones you'll use most often are `PRIMARY_AUTHOR_ROLE` and `AUTHOR_ROLE`.
* `wikipedia_name`: The name of the person's Wikipedia page, e.g. "Stephen_King"
* `lc`: The person's LCCN, e.g. "n79063767"
* `viaf`: The person's VIAF number, e.g. "97113511"
* `biography`: The person's biography, as you might see on an inside book jacket.
* `aliases`: A list of alternate names for this person, e.g. `["Richard Bachman"]`

Of these, the only ones that are really important are `display_name`, `sort_name`, and `roles`. If we have `display_name` we can try to automatically figure out `sort_name`, and vice versa, but we don't always get it right.

## `SubjectData`

This class represents a decision to classify a book under one subject or another.

* `type`: The classification scheme in use. The `Subject` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) 
   defines a number of industry standard types such as `DDC` and `BISAC`, as well as  
   types that are specific to third party data providers like `OVERDRIVE`. The catch-all `TAG` is designed to hold 
   unstructured tags.
   
   If your data source defines its own classification scheme, you can add a constant for it in `Subject`. You'll 
   also need to add a`Classifier` subclass to [core/classifier.py](https://github.com/NYPL-Simplified/server_core/blob/master/classifier.py) 
   to boil down the classifications into the genres defined by Library Simplified.
* `identifier`: The identifying string for the subject in use, e.g. "032" for Dewey Decimal Classification 032.
* `name`: The human-readable name associated with the subject, if different from the identifier. For example, the `identifier` of Dewey Decimal Classification 032 is "032" and the `name` is "Encyclopedias in English".
* `weight`: A number indicating how strongly you believe this book should be classified under this subject.
   
   There are no hard and fast rules here, but in general this number should range from 1 to 100, with 100 meaning 
   that you trust this data a lot, and 1 meaning that it's just a suggestion. This number can have different values
   for different classifications from the same data source. We weight Overdrive classifications very highly,
   but we give very little weight to the freeform tags provided by Overdrive.

## `IdentifierData`

This class represents an identifying string used to distinguish this book from other books.

* `type`: The `Identifier` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) 
   defines a number of standard types such as `ISBN` and `URI`, as well as  
   types that are specific to third parties like `OVERDRIVE_ID` and `ASIN`. You can add an identifier type by adding 
   a constant to this class.

* `identifier`: The identifying string itself, e.g. "9781453219539" or "019f21e3-9de9-4c40-95a4-dfabf55e7801" or "https://standardebooks.org/ebooks/miguel-de-cervantes-saavedra/don-quixote/john-ormsby/".

* `weight`: On a scale of 0 to 1, how certain are you that this string identifies the book whose `Metadata` you're 
  putting it into?
  
  If this is true by definition, 1 is an appropriate value. If Overdrive says that the Overdrive ID 
  of a book is "019f21e3-9de9-4c40-95a4-dfabf55e7801", then it's "019f21e3-9de9-4c40-95a4-dfabf55e7801", by 
  definition. Otherwise it's a judgment call, depending on how much you trust the data. If you guessed at an 
  identifier based on a title/author match, then this number won't be terribly high.
  
  Most commercial data sources include a proprietary ID as well as an ISBN for the same book. In this case it's
  appropriate to assign 1 to both identifiers, but the proprietary ID should be used as the primary identifier.
  
  For a book's `primary_identifier`, this value must be 1.

## `FormatData`

This class represents a format in which the book is available. We consider 'format' to cover both the type of the file and the DRM hoops one must jump through to get the file. 

* `content_type`: The media type of the file or files that contain the book.  The `Representation` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) defines a number of constants for common media types like `EPUB_MEDIA_TYPE`.
* `drm_scheme`: The DRM scheme used to encrypt or obscure access to the book. The `DeliveryMechanism` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) defines a number of constants for common DRM schemes like `ADOBE_DRM`.
* `link`: A `LinkData` object representing an actual freely-available copy of the book in this format. If you put a `LinkData` in here, there's no need to also include that object in `Metadata.links`.
* `rights_uri`: The license under which this book is offered to the general public. The `RightsStatus` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) defines constants for the various Creative Commons licenses and a few others. The default is `RightsStatus.IN_COPYRIGHT`, indicating a book that is not available to the general public except through special arrangement.

Note that adding a new `content_type` or `drm_scheme` will not make the system _support_ the underlying content type or DRM scheme. It merely allows the system to record the _fact_ that a third party has made a book available in a certain format and through a certain DRM scheme.

## `LinkData`

This class represents a document associated with this book; usually a cover image, a description, or an electronic copy of the book. The assumption is that this file can be downloaded by anyone without authentication, and opened up without any special tools. Which is to say, this is not how DRM-encrypted books are delivered.

* `href`: The URL to the file.
* `media_type`: The media type of the file.
* `content`: In lieu of a URL, you can provide the actual content of a file. This is most often used for textual descriptions of a book.
* `rel`: The relationship between the book and the document, or 'link relation'. The `Hyperlink` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py) defines a number of link relations, the most important being `IMAGE` (for a cover image), `DESCRIPTION` (for a textual description), and `OPEN_ACCESS_DOWNLOAD` (for a freely available copy of a book).
* `thumbnail`: If this is an image, and you also know the URL of a smaller version of the image, you can create a `LinkData` object for the smaller version and make it the `thumbnail` of the full-size image.

## `MeasurementData`

Sometimes data sources take measurements of a book's quality, popularity, reading level, or some other numeric value. These values can go into `MeasurementData` objects.

You can store as much data here as you want, but to get it to actually _do_ something you'll need to change one of the methods in the `Measurement` class in [core/model.py](https://github.com/NYPL-Simplified/server_core/blob/master/model.py). This stuff has to be calibrated. To take an obvious example, Amazon's "sales rank" and Overdrive's "popularity" both measure the same quantity (popularity), but a low sales rank means a book is very popular while a low Overdrive popularity means the opposite.

* `quantity_measured`: The quantity being measured. Some popular measurements are defined in the `Measurement` class.
  Again, adding a new type of measurement (or even an existing measurement from a new source) won't do anything.
  The measurement values have to be calibrated, which requires changing the `Measurement` class.
* `value`: The value of the measurement.
* `weight`: How confident you are in the accuracy of the measurement, relative to other measurements of the same quantity from the same source. This should be a number from 0..1.
* `taken_at`: The date at which the measurement was taken. In general, only the most recent measurement is considered.

# Writing to the database

When you create a `Metadata` or `CirculationData` object, you've captured the current state of affairs as described on some remote server. Now you need to make the local database reflect that state of affairs. The `apply` method will take your metadata-layer object and create or modify the necessary database objects to make the database reflect what the third party says.

## `CirculationData.apply(self, _db, collection)`

* `collection` is a `Collection` object representing the collection that contains the book you're talking about. If necessary, `apply` will create a `LicensePool` in that collection representing the fact that this book is available in this collection.
* `replace` is a `ReplacementPolicy` object (see below).

## `Metadata.apply(self, edition, collection)`

* `edition` is an `Edition` object representing bibliographic information about this book. You can get an appropriate edition by calling `edition(_db)` on the `Metadata` object.
* `collection` is a `Collection` object representing the collection that contains the book you're talking about. If you're talking about the book in abstract terms, rather than any particular copies of the book, you can leave this blank.

## `ReplacementPolicy`

When you call an `apply` method, you're supposed to pass in a
`ReplacementPolicy` object as well. This object sets the rules for when a value _overrides_ a previously existing value in the database, versus when it _extends_ previously existing values in the database.

In practice, the details don't matter much. When you're dealing with actual copies of the books (and thus, with `CirculationData`), you probably want to use `ReplacementPolicy.from_license_source`. When you're just editing information about a book (so you have a `Metadata` but no `CirculationData`), you want to use `ReplacementPolicy.from_metadata_source`.

TBD


