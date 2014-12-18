#3M Intercept HOWTO

1. Get yourself an installation of 3M's Cloud Library application for iOS. The desktop version does things differently and does not seem to allow this approach to work, so don't try it!

2. Step up an HTTP proxy to intercept requests from the application. The easiest way to do this is to download and run Charles (http://www.charlesproxy.com) on a computer that's on the same network as your iOS device.

3. Configure your device to use an HTTP proxy via Settings -> Wi-Fi. Set the HTTP proxy configuration to Manual, use your computer's IP for the Server field, and use 8888 -- the default port Charles uses -- for the Port field. Leave authentication off.

4. Open the 3M app and find a book you want to borrow.

5. In Charles, add a breakpoint (Proxy -> Breakpoints...) for requests to the following URL:

    http://ebookfs.3m.com/fulfillment/Fulfill

6. Make sure breakpoints are enabled in Charles.

7. Hit the Borrow button in the 3M app. Moments later, the request should appear in Charles.

8. In the Session tab of Charles, find the request and examine the response. You should see something like the following:

    http://paste.lisp.org/display/144796/raw

9. The fulfillmentToken entity is what we're after. Copy it and save it in a new file with a ".acsm" file extension.

10. Open the ACSM file in Adobe Digital Editions. The ACSM file will be fulfilled with whatever authorization information is currently stored. Critically, that authorization need not have anything to do with 3M at all! It can be an OverDrive login, a Datalogics test login, or in fact any account associated with any other vendor ID.
