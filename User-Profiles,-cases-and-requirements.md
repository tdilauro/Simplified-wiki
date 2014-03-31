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

        There doesn't seem to be built in support for this, but adding it should be relatively trivial. **JF** 

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
### General Use Cases ###
_As a_ **New User/Existing User** 
_I want to_

1. Connect to my 3rd party reading list (Google Bookshelves, Good Reads, etc..)
	
_So I can_ **See my previous reads**	

_Confirmation Criteria_

* The App can connect to my Google Books Reading List
* Use my Good Reads books shelf API

2. Connect to my 3rd party reading list (Google Bookshelves, Good Reads, etc..)
	
_So I can_ **Record my reads**

_Confirmation Criteria_

* The App can transact on my Google Books Reading List
* The App can transact on my Good Reads books shelf API

3. Create Wish List

_So I can_ **So I can get notifications when the book is available**

_Confirmation Criteria_

*The App can use my Google Books Reading List to identify availability in the catalogue
*The App can use my Good Reads books shelf API to identify availability in the catalogue

4. See primarily titles that are available

_So I don't have to browse through titles not available to read now_

_Confirmation Criteria_

*I get Titles I can download now 
*I get Titles in the format my device and app supports
*I get only titles that have an available copy (OD, 3M, Haithi, Guttenberg, other..)

5.  Get a physical copy if the eBook isn't available

_So I don't have to wait to read the book if format is not an issue_

*I can place a hold on the physical format if the digital format is not available
*The physical title copy is available for pick-up

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


