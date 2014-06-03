We have three complementary jobs when it comes to presenting books:

* To classify the books into feeds, such that each book in a feed is related in some way.
* To show each patron the feeds most likely to interest them. 
* Within each feed, to put more the best and most interesting books at the front of the feed.

The first is a job for a classification scheme. The second and third are jobs for a recommendation engine. 

A good classification scheme reduces the need for a good recommendation engine, and vice versa. The more precisely you know what _kind_ of book someone likes, the less picky you need to be about the relative ranking of books in that category. But if you could always present someone with the perfect book they want right now, then classification would be irrelevant.

# Classification schemes

A lot of schemes have been devised to classify books.

* [BISAC](https://www.bisg.org/complete-bisac-subject-headings-2013-edition) Sample: "POLITICAL SCIENCE / Public Policy / City Planning & Urban Development"

* [FAST](http://www.oclc.org/research/activities/fast/download.html) A faceted classification of BISAC headings. Sample: "African American teenagers--Education" Full dataset available under CC-BY.

* [BIC](http://editeur.dyndns.org/bic_categories) Sample: "FKC" (Classic horror and ghost stories), child of "FK" (Horror and ghost stories), child of "F" (Fiction). A bidirectional BIC↔BISAC mapping can be [obtained from BISG](https://www.bisg.org/news/bisg-bulletin-extraupdated-bic-bisac-subject-codes-mapping-available-now).

* [Dewey Decimal classification](http://dewey.info/) Sample: "188" (Stoic philosophy), child of "18" (Ancient, medieval & eastern philosophy), child of "1" (Philosophy & psychology)

* [Library of Congress classification](http://www.loc.gov/catdir/cpso/lcco/) Sample: "QE521-545" (Volcanoes and earthquakes), child of "QE" (Geology), child of "Q" (Science)

* [Library of Congress subject headings](http://www.loc.gov/aba/cataloging/subject/) Sample: "Fundraising cookbooks"

* Bookstores like Amazon have their own proprietary classifications, e.g. "Books > Arts & Photography > Architecture > Urban & Land Use Planning". These also show up as GoodReads "genres".

* Amazon books also come with classifications from the publishers, although these aren't displayed very prominently. Major publishers tend to use LOC subject headings or BISAC classifications. Self-published books effectively use tags.

* When you join GoodReads, it asks you to select your favorite genres. Here are the available genres:
 - Art
 - Biography
 - Business
 - Chick-lit
 - Children's
 - Christian
 - Classics
 - Comics
 - Contemporary
 - Cookbooks
 - Crime
 - Ebooks
 - Fantasy
 - Fiction
 - Gay and Lesbian
 - Graphic novels
 - Historical fiction
 - History
 - Horror
 - Humor and Comedy
 - Manga
 - Memoir
 - Music
 - Mystery
 - Non-fiction
 - Paranormal
 - Philosophy
 - Poetry
 - Psychology
 - Religion
 - Romance
 - Science
 - Science fiction
 - Self help
 - Suspense
 - Spirituality
 - Sports
 - Thriller
 - Travel
 - Young-adult


* A book's author and the series it belongs to groups it with other books in a very basic way.

* Folksonomic classifications are emergent classifications derived from use. For instance, GoodReads "shelves", Twitter's hashtags, or the tags in any system that supports tagging. A tag like "hugo-winner" or "female-protagonist" divides the universe of books into two sets: a set of books that have a certain feature and a set of books that lack that feature. Tags may also express private sentiment ("4-stars").

* Similarly, any list of books, no matter what other theme it might have, divides the universe of books into two sets: the books on the list and the books not on the list.

* Another means to classify books as the collection grows would be the following:

#### ART, ARCHITECTURE, & DESIGN
* Architecture
* Art
* Criticism & Theory
* Design
* Fashion
* History
* Photography

#### BIOGRAPHY & MEMOIR

#### BUSINESS & ECONOMICS
* Economics
* Management & Leadership
* Personal Finance & Investing
* Real Estate

* CHILDREN

#### CLASSICS & POETRY
* Classics
* Poetry

#### CRAFTS, COOKING, & GARDEN
* Antiques & Collectibles
* Bartending & Cocktails
* Cooking
* Crafts, Hobbies, & Games
* Gardening
* Health & Diet
* House & Home
* Pets
* Vegetarian & Vegan

#### CRIME, THRILLERS, & MYSTERY
* Action & Adventure
* Espionage
* Hard Boiled
* Legal Thrillers
* Military Thrillers
* Mystery
* Police Procedurals
* Supernatural Thrillers
* Thrillers
* True Crime
* Women Detectives

#### CRITICISM & PHILOSOPHY
* Language Arts & Disciplines
* Literary Criticism
* Philosophy

#### FICTION GENERAL

#### GRAPHIC NOVELS & COMICS
* Literary
* Manga
* Superhero

#### HISTORICAL FICTION

#### HISTORY
* Africa
* Ancient
* Asia
* Civil War
* Europe
* Latin America
* Medieval
* Middle East
* Military
* Modern
* Renaissance
* United States
* World

## Recommendation sources

We only acquire a paid ebook if a librarian thinks it's worth stocking. This sets a minimum level of quality on our paid selection. This means automated techniques for detecting _quality_ aren't as useful. (They may still be useful on our public domain material.)

Recommendations should focus on identifying aspects of a book most likely to appeal to particular audiences, so that we can order the same feeds differently for different audiences. 

#### Goodreads
Let users who are member of Goodreads connect to their Goodreads accounts, and access the books in their shelves, their ratings, their reviews, and their friends – the social reading graph.
* [Good Reads API](https://www.goodreads.com/api)

#### LibraryThing
[LibraryThing API](https://www.librarything.com/services/)

Bibliocommons lists

Reviews (sentiment + subject matter)

## Recommendation engines

http://blog.mortardata.com/post/82195614895/giving-away-our-recommendation-engine-for-free