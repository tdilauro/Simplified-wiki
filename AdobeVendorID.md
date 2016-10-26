A Library Simplified circulation manager can implement the Adobe Vendor ID protocol to act as an issuing authority for Adobe IDs. If you are providing a custom mobile app, and you want to provide Adobe Vendor IDs, you will need to pay Adobe for a new Vendor ID. This can be quite expensive.

Fortunately, if your circulation manager serves patrons through the SimplyE mobile app, all you need to do is connect your circulation manager to the Adobe Vendor ID server run by NYPL. NYPL will issue Adobe Vendor IDs to your patrons on your behalf.

You can choose to do nothing. Your circulation manager will work fine if Vendor ID is not configured. If you don't have any books that are encrypted with Adobe DRM, doing nothing is the smart move--your patrons will never need an Adobe ID. But if you _do_ have books that are encrypted with Adobe DRM, and you choose not to configure a Vendor ID server, your patrons will probably not be able to open books across devices.

# Your own Vendor ID server

This is the expensive option. You will need to acquire a vendor ID and a node value from Adobe. Then you need to put those values in your circulation manager configuration.

Your config file must define an integration called `Adobe Vendor ID`. It must define the following 

* `vendor_id`: This is the name of your vendor ID as registered with Adobe.
* `node_value`: This is the node value supplied by Adobe.

Example:

```
"integrations" : {
	"Adobe Vendor ID" : {
	    "vendor_id" : "My Library",
	    "node_value" : "49aa62d328e2"
	}
}
```

Tell Adobe that your Vendor ID base URL is the root of your circulation manager plus `AdobeAuth`. For example, if your circulation manager is hosted at `https://my-circulation-manager.com/`, your Vendor ID base URL will be `https://my-circulation-manager.com/AdobeAuth`.

Once you set this up, Adobe will certify your implementation and you'll be able to issue Vendor IDs.

# Connect to the SimplyE Vendor ID server

This is the cheap option, but you can only choose it if you're using SimplyE as your OPDS client. In this option you are delegating the job of issuing Adobe IDs to NYPL.

* `vendor_id`: This is the literal string "NYPL". You are getting Adobe IDs from NYPL and not some other source.
* `library_uri`: A URI that represents your library. The URL to your library's website is fine. This lets NYPL distinguish between your patrons and another library's patrons.
* `secret`: A secret string that is shared between you and NYPL. This lets NYPL verify that a request actually came from you and is not someone trying to hack the system.

```
"integrations" : {
	"Adobe Vendor ID" : {
	    "vendor_id" : "NYPL",
            "library_uri" : "http://my-library.org/",
            "secret" : "29789882ff0ea36dc7bdf15232f3021e"
	}
}
```

## How it works

The technical details are in the [[Vendor ID Service spec|https://docs.google.com/document/d/1j8nWPVmy95pJ_iU4UTC-QgHK2QhDUSdQ0OQTFR2NE_0/edit#heading=h.jxwemo85jady]] and the [[DRM Extensions to OPDS|https://github.com/NYPL-Simplified/Simplified/wiki/DRMAutodiscoverySpecs#drm-extensions-to-opds]], but here's roughly how it works:

1. A patron using SimplyE tries to open a book encrypted with Adobe DRM. They don't have an Adobe ID on this device (though they may have created one from another device). They need their Adobe ID.
2. Fortunately, your circulation manager has forseen this possibility and provided a special message that the client can present to Adobe to get an Adobe ID. The message is signed with your library's `secret`. It says: "Patron X at library Y wants their Adobe ID. Signed, library Y." 
3. Adobe receives the message and passes it on to NYPL.
4. NYPL receives the message and is able to verify the signature because you shared your `secret` with NYPL.
5. NYPL creates (if necessary) or looks up (if possible) the Adobe ID for patron X, and sends it back to Adobe.
6. Adobe forwards the Adobe ID back to your patron's SimplyE client.
7. Now that they have their Adobe ID, the patron is able to open the encrypted book.

In step 2, "X" is a made-up alias for the patron, not the patron's actual identifier as kept in your ILS. NYPL can distinguish between one patron and another, but has no way of tying "X" back to a specific library card. Adobe sees the message as well, but they also have no way of tying "X" back to a specific person.

In step 4, NYPL knows that the message is from library Y because it was signed with library Y's secret. 