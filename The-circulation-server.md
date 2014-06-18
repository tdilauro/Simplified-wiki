The circulation server is responsible for a _consolidated view_ of every
book in our catalog (whether we own that book through open-access,
through a license, or both). For each book, this view includes:

* The _bibliographic information_ we want to display to the user,
  e.g. title, author, language, audience.

* A numeric estimate of the book's quality.

* An assignment of the book to one (and only one) lane.

* Information about each active license for the book.

* Information about all current activity between Simplified patrons
  and the book.

The circulation server is not necessarily responsible for gathering,
keeping, or processing the raw data associated with this consolidated
view (e.g. OCLC Classify records, NYT bestseller information, or
Amazon reviews).

Hoever, the circulation server is responsible for the _search engine_,
which for now probably means storing that raw data. Search results are
served as OPDS feeds. The only supported search is a combination
title/author search.

The circulation server _serves OPDS feeds of books_ written in various
languages and found in various lanes. It can order these feeds by
title and author. It can also generate 'recommended' feeds which
include a random sampling of high-quality books for a given
combination of language+lane.

The circulation server is responsible for _knowing about and granting
access to binary files_ such as cover images, open-access books,
DRM-encrypted books, unencrypted previews, and license files. It is
not responsible for hosting any of those files; it just knows where
they are.

In particular, the circulation server is responsible for acting on behalf
of a patron to _orchestrate the process of obtaining a license file_
for a DRM-encrypted book, regardless of where the license server is
located, who controls the license server, or which type of DRM has
been applied to the book. The circulation server is responsible for
telling the client where to go to get the license file and the
DRM-encrypted book, but not necessarily responsible for hosting those
files.

The circulation server is responsible for processing user requests for
returning a book early, extending a license, or joining the hold queue
for a book.

The circulation server is responsible for _validating a patron's barcode
and PIN_ with Millenium or with any successor SSO service. It is not
responsible for storing a patron's barcode or PIN.