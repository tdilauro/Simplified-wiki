# Step 1: Compare how ereader is set up on SimplyE (using R1) and on the R2-test-app

In SimplyE currently the only class that actually depends on R1 is NYPLReaderReadiumView. This is a massive view that contains the WebView presenting the book, as well as logic and hacks to interact with the ePub, including injecting JS into it. It holds a Container that provides C++ objects ("Packages", wrapped by a relative Objective-C++ class), used to get HTML files for each chapter from disk, using a HTTP server running on the client.

R2 approach is much more structured: 
1. the test-app has some light infrastructure to essentially give a ePub URL to R2's parser. The URL can be a file or http URL. 
2. R2 parsing APIs return a R2Shared::Publication object. 
3. The test app then adds the Publication to its R2Streamer::PublicationServer (running locally).
4. It also creates a EPUBViewController. This is a custom light-weight VC that wraps a R2Navigator::EPUBNavigatorViewController for handling ebook nav commands (turn pages, etc), as well as other custom events such as presenting user settings, TOC, etc.  
    - The EPUBNavigatorViewController does all the heavy lifting of setting up the view to render the book contents for us, which is great, leaving corollary aspects of the UI to the client app (i.e. the light-weight VC). It can also handle the backend storage of the user settings.

# Step 2: Integrate basic backend functionality

While R2 has good [reference documentation](https://github.com/readium/architecture), there's no "Getting Started" guide. They do provide a test-app, which can be used as a base model. So the first step was to import the core non-UI components, setting things up to the point of at least parsing a book into a Publication object.

One gotcha is that the main "Book" models between SimplyE and the R2-test-app are (obviously) different. So there are some assumptions that need to be converted over. Writing some quick extensions to our NYPLBook model in order to adapt our model to their infrastructure seems sustainable, although further refactoring will likely be needed.

The main idea here was, if we can get a Publication object out of R2, all we have to do is convert that model to our reader. This however quickly appeared as a daunting task.  Our eReader seems very tightly coupled with R1, not to mention the hacks we'd have to understand / support.

# Step 3: Bring in the Navigator

A significant part of R2's offerings is the [R2Navigator framework](https://github.com/readium/architecture/tree/master/navigator). This solves one of the biggest problems, which is rendering the book and dealing with the webview. The good thing is that R2Navigator still allows the client app to define how the chrome should look like. This is what the test-app does: its UI code lives on top of the actual ebook renderer. For instance, the test-app can configure the navigation bar in the reader, as well as the bottom bar. The user settings configuration and the TOC are separate VCs, local to the sample app, therefore easily modifiable.

# Further Integration

These steps seem in line with the way R2 is designed to work. While we have a requirement to configure the new reader to look like the old reader from a user POV, this seems possible with what we've seen so far.

Current state: https://github.com/ettore/Simplified-iOS/tree/story/SIMPLY-2489/R2-integration

TODO items:
- Implement switch between R1 and R2 ereader depending if book has DRM or not
- Sort top icons (TOC, settings, bookmark) in same order as R1
- Refactor R1 reader settings view in a way that’s reusable
    - e.g. so that it’s possible to wire the buttons to different controllers
- Make R2 reader settings view open from bottom as in R1
- On iPhone, fix R1 way of presenting user settings (present on sheet instead of just adding view a subview)
- On iPhone, consider if we can open settings as on iPad (from top dropdown instead of bottom) to simplify code
- Wire add bookmark for R2
- Fix resume reading (implement NYPLBook.locator getter)
- Style bottom view (page # - chapter) as in R1