_(NOTE: wording is not final)_

[Clickable version of this mockup](https://popapp.in/projects/53738be686797f32740ef81a/preview)

## Available everywhere (except the reader)
- [ ] [Settings](#app-settings)
- [ ] [My books](#my-books)
- [ ] [Library](#library)

## Library
- [ ] Notification of items waiting for action (links to My books)
- [ ] Lanes, each with:
  - Name
  - n book covers
  - ["More" button](#lane-detail)
  - "Swipe up" action to add book to [ignore list](#ignored-books)
- [ ] [Search bar](#search)
- [ ] NYPL Logo

## Search
- [ ] Keyboard
- [ ] Keyword input area
  - Watermark prompt of e.g. "Title/Author"
- [ ] Book result list, each with:
  - Title
  - Author
  - Cover
  - [Status of book](https://github.com/NYPL/iOS-Reader/wiki/Information-Architecture#client-book-states)
  - "Remove from saved" button (if applicable _or "Forget"_)
  - "Remove hold" button (if applicable)
  - Estimated time until checkout available (if applicable)
  - "Checkout book" button (or "Borrow" if applicable, prompts download)
  - ["Read book" button](#reader) (or simply "Read" if applicable)
  - ["View detail" button](#book-detail or "Details" )

## Lane detail
- [ ] Lane name
- [ ] Option to sort by: Title | Author
- [ ] List of books, each with:
  - Title
  - Author
  - Cover
  - [Status of book](https://github.com/NYPL/iOS-Reader/wiki/Information-Architecture#client-book-states)
  - "Ignore book" button (prompts confirmation)
  - "Save for later" button
  - "Checkout book" button (or "Borrow" if applicable, prompts download)
  - ["View detail" button](#book-detail)
- [ ] [Search bar](#search)
- [ ] Button to return to previous screen

## My books
- [ ] List of books, each with:
  - Title
  - Author
  - Cover
  - [Status of book](https://github.com/NYPL/iOS-Reader/wiki/Information-Architecture#client-book-states)
  - "Remove from saved" button (if applicable)
  - "Remove hold" button (if applicable)
  - Estimated time until checkout available (if applicable)
  - "Checkout book" button (if applicable, prompts download)
  - ["Read book" button](#reader) (if applicable)
  - ["View detail" button](#book-detail)
- [ ] Filter between: All | Saved for later

## Book detail
- [ ] Cover
- [ ] Name
- [ ] Author
- [ ] Abstract
- [ ] ["Read book" button](#reader) ( or simply "Read" if applicable)
- [ ] "Check out" button ( or "Borrow" if applicable, prompts download)
- [ ] "Hold book" button (if applicable)
- [ ] "Return book" button (or "Return" if applicable)
- [ ] "Save for later" button (if applicable)
- [ ] Genre/LCSH headings type of content
- [ ] Estimated time until checkout available (if applicable)
- [ ] "Book due in" message (if applicable)
- [ ] Button to return to previous screen

## Reader
† Appears on tap or on book end
- [ ] Book length visualization
- [ ] Current position within book length
- [ ] Book text
- [ ] † [Bookmark widget](#bookmark-widget)
- [ ] † [Font/color options](#reader-settings)
- [ ] † "Table of contents" button
- [ ] † "Next page" button/swipe
- [ ] † "Previous page" button/swipe
- [ ] † "Exit reader" button

## Reader settings
- [ ] Choose between font: Sans-serif | Serif
- [ ] Choose between contrast modes: Black-on-white | Sepia | White-on-black
- [ ] "Increase font size" button
- [ ] "Decrease font size" button

## Bookmark widget
- [ ] Page # (or Chapter #)  If applicable, ebooks are flowable and not broken into pages, just resources since the font and screen is variable and the pagination goes away.
- [ ] Text snippet (a short blurb representative of the page being bookmarked... maybe the phrase at the top left corner?)
- [ ] "Remove bookmark" button
- [ ] "Go to bookmark" button

## Book end
- [ ] "Return book" button
- [ ] "Rate book" buttons (5 stars)
- [ ] "Previous page" button/swipe

## App settings
- [ ] Barcode
- [ ] Barcode numbers
- [ ] PIN (obscured => tap to reveal?)
- [ ] ["Ignored books" button](#ignored-books)
- [ ] ["Feedback" button](#feedback)
- [ ] ["Credits/Acknowledgments" button](#credits)
- [ ] "Logout" button (prompts confirmation of data deletion)

## Ignored books
- [ ] "Ignored books" title text
- [ ] List of books, each with:
  - Title
  - Author
  - Cover
  - "Remove from ignored" button
  - ["View detail" button](#book-detail)

## Feedback
- [ ] Text area for feedback
- [ ] Text area for email (optional)
- [ ] "Send feedback" button
- [ ] "Cancel" button

## Credits
- [ ] Credits text
- [ ] Acknowledgments text
- [ ] Button to return to previous screen

## Authentication
- [ ] Message indicating requirement to log in
- [ ] Barcode input
- [ ] Button to prompt camera for automatic barcode scan
- [ ] PIN input
- [ ] "Sign in" button
- [ ] "Cancel" button
- [ ] "Sign up" button **(TO BE CONFIRMED)**