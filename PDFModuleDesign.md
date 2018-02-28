### Rough outline of design for providing PDF support

Overall Diagram of Architecture ([Slack: Original Discussion](https://librarysimplified.slack.com/archives/G96PHK59Q/p1518196602000323)):
```
 --------------  \     \
| SimplyE App |   |     | P
 --------------   | O   | U
       |          | U   | B
 --------------   | R   | L
| PDF Renderer |  |     | I
|   Provider   |  | C   | C
 --------------   | O  /
       |          | D
 --------------   | E  \  P
|   PSPDFKit   |  |     | R
|   Provider   |  |     | I
 --------------  /      | V
       |                | A
 --------------         | T
|   PSPDFKit   |        | E
 --------------        /
```

The current implementation of the module/library is `PDF` in the iOS project, and that would need to be split into TWO separate libraries (eventually): `PDFRendererProvider` and `PSPDFKitProvider`. I'll kind of outline below how this will look.

The main idea is looking at what is currently in `PSPDFKit` and asking ourselves "what would these methods look like if we didn't know that PSPDFKit existed?"

So instead of what's currently in the sample/SimplyE app:
```
let pdfViewController = PDFViewController.init(documentURL: fileURL, openToPage: book.lastPageRead, bookmarks: book.bookmarks, annotations: book.annotations, PSPDFKitLicense: APIKeys.PDFLicenseKey, delegate: self)
self.navigationController?.pushViewController(pdfViewController, animated: true)
```

it would actually be something like:
```
let PDFRenderer = PDFRendererProvider(document: URL, page: UInt = 0, bookmarks: [BookmarksClass], annotations: [AnnotationsClass], delegate: PDFDelegate)
let pdfViewController = PDFRenderer.createViewController() //This may be unnecessary if PDFRenderer creates or returns a viewController immediately.
self.navigationController?.pushViewController(pdfViewController)
...
```

And the main thing is that now, all the code you have written is still valid beacuse it ties into PSPDFKit, but it belongs in `PSPDFKitProvider` with the one change that all those methods are now implementations of a NEW Protocol (interface):

```
Protocol PDFRendererProviderDelegate {
  // Functions that must be implemented by "PSPDFKitProvider"
  func createViewController() -> UIViewController
  func getAllAnnotations() -> [AnnotationsBaseClass] //Just an idea...
  func documentWillClose(onPage: Int) -> Void
  ...many more
  // Variables that must be provided by "PSPDFKitProvider"
  var currentPage: Int
  ...

}
```

And now `PSPDFKitProvider` should implement (or extend) this protocol:

```
import PSPDFKit
...
class PSPDFKitProvider: PDFRendererProviderDelegate {
  
  override func createViewController() -> UIViewController {
    //Now we "know" about PSPDFKit, so a variation of what's already been written goes here:
    let vc = PSPDFViewController.init(documentURL: URL, openToPage page: UInt = 0, bookmarks pages: [UInt] = [], annotations annotationsData: [Data] = [], PSPDFKitLicense: String, delegate: PDFViewControllerDelegate?)...
    return vc
  }

  ...override all the other funcs and variables in the protocol

}
```


Now moving on to Annotations or Bookmarks, the concept is very similar.
Starting with the simplest example, bookmarks. We know that PSPDFKit just uses Int's, BUT some other SDK may use a String (who knows!).. So in `PDFRendererProvider` we must create a protocol for Bookmarks:

```
protocol BookmarkDelegate {
  createBookmark() -> BookmarkClass
}

class BookmarkClass {
  var pageIndex: Int
}
```

So inside `PSPDFKitProvider` or someother a related class that extends, or conforms to the protocol, it'd be simple since it also uses an Int to represent the page:
```
override func createBookmark() -> BookmarkClass {
  let page = PSPDFViewController.pageIndex
  let bookmark = BookmarkClass.init(page)
  return bookmark
}
```
some time later we may use some other renderer that could save pages as a string, or a float..
```
override func createBookmark() -> BookmarkClass {
  let pageString: String = pdf.JS.currentPageProperty
  let pageInt: Int = pageString.convertStringToInt()
  let bookmark = BookmarkClass.init(pageInt)
  return bookmark
}
```


## Example of dependency between the modules

```
// Main App

import RendererProvider

class PDFManager {

  let pdfRenderer = RendererProvider.init() as ViewController

  pdfRenderer.openPDF(pdfFileUrl)
  let currentPage = pdfRenderer.getCurrentPage()
  ...
}

// RendererProvider module (public)

import PSPDFKitRenderer

protocol RendererProviderProtocol {
  var currentPage: Int
  func saveBookmarks()
  func openPDF(fileUrl: Url)
  ...etc.
}

class RendererProvider {

  var renderer: RendererProviderProtocol

  init() {

    self.renderer = PSPDFKitRenderer.init()

  }

  public func openPDF() { 
    renderer.openPDF()
  }

  public func getCurrentPage() {
    renderer.currentPage()
  }

}


// PSPDFKitRenderer Module (private)

import RendererProvider
import PSPDFKit

class PSPDFKitRenderer: RendererProviderProtocol {

  override var currentPage: Int? {
   get {
    return pspdfkit.currentPage() //whatever PSPDFKit does...
   }
  }

  init() {

    self.pspdfkit = PSPDFViewController.init() //whatever PSPDFKit does...

  }

  override func saveBookmarks() {

    PSPDFKitCall.bookmarks()

  }

  override func openPDF(fileUrl) {

    PSPDFKitCall.openPDF(fileUrl)

  }

}
```