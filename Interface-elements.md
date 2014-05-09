_(NOTE: wording is not final)_
## Available everywhere (except the reader)
- [ ] [Settings](#settings)
- [ ] [My books](#mybooks)
- [ ] [Library](#library)

## <a name="library"></a>Library
- [ ] Notification of items waiting for action (links to My books)
- [ ] Lanes, each with:
  - Name
  - n book covers
  - ["More" button](#lanedetail)
  - "Swipe up" action to add book to [ignore list](#ignore)
- [ ] [Search bar](#search)

## <a name="search"></a>Search

## <a name="lanedetail"></a>Lane detail
- [ ] Lane name
- [ ] Option to sort by: Name | Author
- [ ] List of books, each with:
  - Title
  - Author
  - Cover
  - [Status of book](https://github.com/NYPL/iOS-Reader/wiki/Information-Architecture#client-book-states)
  - "Ignore book" button (prompts confirmation)
  - "Save for later" button
  - ["View detail" button](#bookdetail)
- [ ] [Search bar](#search)
- [ ] Button to return to previous screen

## <a name="mybooks"></a>My books
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
  - ["View detail" button](#bookdetail)
- [ ] Filter between: All | Saved for later

## <a name="bookdetail"></a>Book detail
- [ ] Cover
- [ ] Name
- [ ] Author
- [ ] Abstract
- [ ] ["Read book" button](#reader) (if applicable)
- [ ] "Check out" button (if applicable, prompts download)
- [ ] "Hold book" button (if applicable)
- [ ] "Save for later" button
- [ ] Genre/LCSH headings type of content
- [ ] Estimated time until checkout available (if applicable)
- [ ] Button to return to previous screen

## <a name="reader"></a>Reader
† Appears on tap or on book end
- [ ] Book length visualization
- [ ] Current position within book length
- [ ] Book text
- [ ] † [Bookmark widget](#bwidget)
- [ ] † [Font/color options](#readersettings)
- [ ] † "Table of contents" button
- [ ] † "Next page" button/swipe
- [ ] † "Previous page" button/swipe
- [ ] † "Exit reader" button

## <a name="readersettings"></a>Reader settings
- [ ] Choose between font: Sans-serif | Serif
- [ ] Choose between contrast modes: Black-on-white | Sepia | White-on-black
- [ ] "Increase font size" button
- [ ] "Decrease font size" button

## <a name="bwidget"></a>Bookmark widget
- [ ] Page # (or Chapter #)
- [ ] Text snippet
- [ ] "Remove bookmark" button
- [ ] "Go to bookmark" button

## <a name="bookend"></a>Book end
- [ ] "Return book" button
- [ ] "Rate book" buttons (5 stars)
- [ ] "Previous page" button/swipe

## <a name="settings"></a>App settings
- [ ] Barcode
- [ ] Barcode numbers
- [ ] PIN (obscured => tap to reveal?)
- [ ] ["Ignored books" button](#ignored)
- [ ] ["Feedback" button](#feedback)
- [ ] ["Credits/Acknowledgments" button](#credits)
- [ ] "Logout" button (prompts confirmation of data deletion)

## <a name="ignored"></a>Ignored books
- [ ] "Ignored books" title text
- [ ] List of books, each with:
  - Title
  - Author
  - Cover
  - "Remove from ignored" button
  - ["View detail" button](#bookdetail)

## <a name="feedback"></a>Feedback
- [ ] Text area for feedback
- [ ] Text area for email (optional)
- [ ] "Send feedback" button
- [ ] "Cancel" button

## <a name="credits"></a>Credits
- [ ] Credits text
- [ ] Acknowledgments text
- [ ] Button to return to previous screen

## <a name="auth"></a>Authentication
- [ ] Message indicating requirement to log in
- [ ] Barcode input
- [ ] Button to prompt camera for automatic barcode scan
- [ ] PIN input
- [ ] "Sign in" button
- [ ] "Cancel" button
- [ ] "Sign up" button **(TO BE CONFIRMED)**