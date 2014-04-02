### Basic eReader iOS Application Requirements

***

#### eReader Platform requirements
* iPad
* iPhone
* iOS 6 and 7
* Launch Readium EPUB Engine

#### eReader app requirements
**Basic Requirements**

* Open EPUB2
* Open EPUB3
* Open EPUB and Display Book Jacket- on tap got to Chapter One
        
        See tableView:didSelectRowAtIndexPath: in NavigationElementController. Can be accomplished by adding a tap recognizer on top, and telling it which file to load on that tap.

* Open EPUB and Display Content - Chapter 1

        See tableView:didSelectRowAtIndexPath: in NavigationElementController, it pushes to a specific RDNavigationElement when you tap a cell, to a certain file based on what was specified in the datasource.

* Automatically Size Font percentage equivalent to 12pt Font Calibri iPhone

        Scaling is done on a percentage basis. In launcher app, see EpubSettings - fontScale property.

* Automatically Size Font to equivalent percentage to 14pt Font Calibri iPad

        Scaling is done on a percentage basis. In launcher app, see EpubSettings - fontScale property.

* Automatically set view to single page view on iPhone and iPad.  

        Not all books seem to support two column views. In launcher app, see EpubSettings - isSyntheticSpread property, where YES means two columns, and NO means one column.

* Allow flowable content to be scrolled via single touch swipe

        There doesn't seem to be built in support for this, but adding it should be relatively trivial.  

* Advance page turn after end of paginated content is scrolled (ergo - infinite scroll) 

        I don't see any support for this built in via the current architecture.

* Turn page back with left swipe or double tap to advance page forward
        
        There doesn't seem to be built in support for this, but adding it should be relatively trivial.

***

**View Navigation requirements**

* Top Bar (TB) Menue for Reading View
* **TB:Options icon** - access drop down menu for _Options Menue (OM)_ with following items:
* _OM:Display Options icon_ - access control to increase decrease font, adjust brightness, change front from Sans Serif (Calibre) to Serif (Times Roman), Day Reading (White background, Black Text) or Night Reading (Black Background, White Text)
* _OM:Table of Contents icon_ - on tap opens table of contents control
* _OM:Book Marks icon_ - on tap opens book mark control for given work 
* _OM:Share icon_ - on tap opens native share feature of iOS
* _Add to shelf icon_ - opens my shelf control (see shelf control requirements)
* **TB:Book Mark Page icon** -  
* **TB:Share icon** - on tap opens native share feature of iOS
* **TB:Search Content icon** - on tap opens content search control
* **TB: Back arrow** - on tap navigates back from selected sub menu, control or view, or current content location to book Jacket or BookShelf List or Home screen

**Optional requirements:Botom Bar Menu (BM), Controls**
* Control:Page Number display/counter
* Control:Content Guage (%percent read)
* BM:Book Icon - on tap access content Table of Contents 

***
### General Use Cases

_As a_ **New User/Existing User** 
_I want to:_

**Connect to my 3rd party reading list (Google Bookshelves, Good Reads, etc..)**

***
	
_So I can_ **See my previous reads**	

_Confirmation Criteria_

* The App can connect to my Google Books Reading List
* Use my Good Reads books shelf API

**Connect to my 3rd party reading list (Google Bookshelves, Good Reads, etc..)**

***
	
_So I can_ **Record my reads**

_Confirmation Criteria_

* The App can transact on my Google Books Reading List
* The App can transact on my Good Reads books shelf API

**Create Wish List**

***

_So I can_ **get notifications when the book is available**

_Confirmation Criteria_

* The App can use my Google Books Reading List to identify availability in the catalogue
* The App can use my Good Reads books shelf API to identify availability in the catalogue

**See primarily titles that are available**

***

_So I can_ **browse through titles not available to read now**

_Confirmation Criteria_

* I get Titles I can download now
* I get Titles in the format my device and app supports
* I get only titles that have an available copy (OD, 3M, Haithi, Guttenberg, other..)

**Get a physical copy if the eBook isn't available**

***

_So I can_ **read the book when format is not an issue with me**

_Confirmation Criteria_

* I can place a hold on the physical format if the digital format is not available
* The physical title copy is available for pick-up

**Choose to see titles available to hold or get at a later date**

***

_So I can_ **read the book for free**

_Confirmation Criteria_

* I can reserve (hold) a title via the application
* The title shows up in a holds list
* The title is on hold in the library ILS and OPAC

**Search available titles**

***

_As a User_ **Looking for Something to Read**

_So I can_  **find the title I am looking for and read it now**

**Browse Recommended Titles by Librarians**

_As a User_ **Looking for Something to Read**

_So I can_  **find a title from a trusted source and read it now**

**Browse Recommended Titles that are recommended by Friends**

_As a User_ **Looking for Something to Read**

_So I can_  **find a title from a trusted source and read it now**

**Browse Recommended Titles of genera I am interested**

_As a User_ **Looking for Something to Read**

_So I can_  **find a title from a trusted source for a genera I like and read it now**





## Research and Reference User (R&RU)

**Use Cases**
* As a Researcher and Reference Users I want to conduct a catalog search or category browse
* As a Researcher wants to download books
* is a public library user
* reads the relevant chapters
* Bookmarks catalogue items (books) for borrowing
* wants to capture notes about methods in notes, or Evernote on her mobile phone / tablet when on  a job site

## Leisure and Entertainment (L&EU)
### Commuter

### Bibliophile

## Mother/Care Giver

## Millennial?

## Teen/Young Adult

### Male

### Female


