Most configuration of the Library Simplified circulation manager happens through the administrative interface. Configuration details are stored in a database. But which database?

The circulation manager will look in the environment variable `SIMPLIFIED_PRODUCTION_DATABASE` for a database URL, and will try to connect to that URL to get the rest of the site configuration. To configure your database, make sure some code like this is run before you start up the circulation manager:

```
export SIMPLIFIED_PRODUCTION_DATABASE=postgres://[username]:[password]@[hostname]:[port]/[database name]
```

To run the unit tests, you'll need to set the SIMPLIFIED_TEST_DATABASE variable instead.

```
export SIMPLIFIED_TEST_DATABASE=postgres://[username]:[password]@[hostname]:[port]/[database name]
```

# Web-based configuration

Once you get a circulation manager running, visit the `/admin/` URL, e.g. `http://localhost:6500/admin/`. First, you'll be asked to set up an administrator account. You'll most likely do this by setting an admin username and password. 

From this point onward, you'll need that username and password to set the site configuration.

Once you log in with an admin username and password, you'll be able to create and administer libraries and collections through the web-based admin interface.

# Configuration file-based configuration

In older versions of the software, configuration was done with a JSON configuration file. Instead of setting `SIMPLIFIED_PRODUCTION_DATABASE` to the database URL, you would set `SIMPLIFIED_CONFIGURATION_FILE` to the path to the JSON configuration file.

Currently, you only need to set up a JSON configuration file if you want to do one of these things:

* [[Customize your lane configuration.|LaneConfiguration]]
* Customize the language collections your library offers.
* Restrict which titles certain patrons can borrow.
* Restrict which lanes certain patrons can see.

Otherwise, you should be able to ignore everything below this point. In the long run, the JSON configuration file will disappear altogether.

The configuration file contains all the necessary information to distinguish one Library Simplified installation from another. This file contains a lot of miscellaneous stuff, but the main sections are:

* Links to especially important information that differs between libraries (privacy policy, terms of service)
* Policies that differ between libraries, such as how to authenticate patrons, whether patrons can put books on hold, and how books should be organized in lanes.
* Details on integrating with external pieces of software (such as the database) and third-party services (such as ebook providers)

The circulation manager, the metadata wrangler, and the content server each need their own configuration file, but what goes into that configuration file differs from one component to the other.

The path to the configuration file is controlled by the SIMPLIFIED_CONFIGURATION_FILE environment variable.

# General format

A configuration file is parsed as a JSON object, with one exception. Any line in which the first non-whitespace character is a hash (#) is ignored. This makes it possible to put comments in the configuration file.

Your configuration file will look something like this:

```
{
 # Comments are okay
 ...
 "links" : { ... },
 "policies" : { ... },
 "integrations" : { ... }
}
```
 
The sections below contain summary tables explaining which configuration settings can be used with which Library Simplified components. Here's an example:

| Setting                            | circ | metadata | content |
| ---------------------------------- | ---- | -------- | ------- |
| foo                                | O    | X        |         |

This means that the `foo` setting is required when configuring a metadata wrangler, optional when configuring a circulation manager, and unsupported or irrelevant to a content server.

## Links

The value of `"links"` should be a JSON object that maps variable names to URLs. Example:

```
{
 "links" : {
	"terms_of_service" : "http://www.librarysimplified.org/NYPLEULA",
	"privacy_policy" : "http://www.nypl.org/help/about-nypl/legal-notices/privacy-policy"
 }
}
```

Currently links are inserted into OPDS feeds to give patrons context.

| Variable name                      | circ | metadata | content |
| ---------------------------------- | ---- | -------- | ------- |
| terms_of_service                   | O    | O        | O       |
| privacy_policy                     | O    | O        | O       |

### `terms_of_service`

A link to the Terms of Service document applicable to a user of the system. Obviously the terms of service for a circulation manager (which is used by patrons) will differ from the terms of service for a metadata wrangler (which is used by libraries).

### `privacy_policy`

A link to the privacy policy applicable to a user of the system.

## Policies

The value of `"policies"` is a JSON object mapping variable names to collection policies. Example:

```
{
    "policies" : {
	"lending": {
	    "60": {"audiences": ["Children"]}, 
	    "152": {"audiences": ["Children"]}, 
	    "62": {"audiences": ["Children"]}
	},
	"languages" : {
	    "large_collections" : ["eng"],
	    "small_collections" : ["rus", "chi", "spa"]
	},
	"holds" : "allow",
	"authentication" : "Millenium"
    }
}
```

Currently policies apply only to circulation managers.

| Variable name                      | circ | metadata | content |
| ---------------------------------- | ---- | -------- | ------- |
| admin_authentication_domain        | O    |          |         |
| authentication                     | O    |          |         |
| external_type_regular_expression   | O    |          |         |
| holds                              | O    |          |         |
| lanes                              | O    |          |         |
| languages                          | X    |          |         |
| lending                            | O    |          |         |
| root_lane                          | O    |          |         |
| minimum_featured_quality           | O    |          |         |

### `admin_authentication_domain`

This domain (e.g. "gmail.com" or "nypl.org") defines the permitted domain of admin-enabled email addresses. Used in conjunction with `include_admin_interface` and the `Google OAuth` integration, anyone with a Google-based email with this domain will be able to use the web-based admin interface to change book details.

### `authentication`

If present, this must identify one of the integrations in the `integrations` section. This explains how the circulation manager is supposed to verify a patron's credentials.

If this is not present, it is assumed that the ebook providers will judge whether or not a patron's credentials are valid.

### `external_type_regular_expression`

If present, this must compile to a regular expression with one (and only one) set of groups. Here's an example:

```
"external_type_regular_expression" : "^([HME])",
```

This regular expression is run against a patron's `authorization_identifier` (i.e. library card number) to determine their `external_type`. This allows different `lending` policies (see below) to be applied to different types of patrons.

In the example above, the first character of a patron's `authorization_identifier` marks them as having an `external_type` of "H", "M", or "E". We could then define `lending` policies to restrict different types of patrons to different subsets of the collection.

### `holds`

There are two possible values here. `"allow"` (the default) indicates that patrons can put a book on hold if it's not available. `"hide"` indicates that books disappear from the collection once all copies are checked out, and that patrons are not allowed to put books on hold.

### `lanes`

This is your chance to customize the lanes presented to patrons. You can have as many or as few lanes as you want. Lane configuration is so complicated it has [[its own wiki page|LaneConfiguration]].

### `languages`

A JSON object whose keys are mapped to lists:

* `large_collections`: A list of the languages in which the library has a very large collection--either the single largest collection or all collections larger than 25,000 books. (Required)

* `small_collections`: A list of other languages for which the library has a decent-sized collection (over 1000 books). Any languages not in this list or the `large_collections` list will be grouped together into an 'Other Languages' section. (Optional)

### `lending`

The value of `lending` is a JSON object that puts restrictions on the patron experience based on their `external_type`. If the patron's `external_type` is one
of the strings found in the keys of `lending`, that patron might be restricted to books from the given audiences.

In the example above, `lending` looks like this:

```
 ...
 "lending" : {
   "60": {"audiences": ["Children"]}, 
   "152": {"audiences": ["Children"]}, 
   "62": {"audiences": ["Children"]}
  },
 ...
```

This means that a patron whose `external_type` is "60", "62", or "152" will only
be able to check out children's books. All other patrons will be able to check out any kind of books.

### `root_lane`

This controls which feed the patron sees when they start up the application (and access the root URL). By default, patrons see the root feed of the collection. Configuring `root_lane` allows you to restrict patrons of different `external_type`s to different subsets of the collection.

In this example, patrons with an `external_type` of `H` are sent to the `Young Adult` collection immediately upon loading the application.

```
 ...
 "root_lane" : {
   "H": "Young Adult", 
   "M": "Middle Grades",
   "E": "Early Grades"
  },
 ...
```

If `root_lane` is configured, patrons will be required to authenticate before accessing the circulation manager--otherwise the circulation manager has no way to determine their `external_type`.

### `minimum_featured_quality`

By default, books are considered for inclusion in lanes of featured books if their quality >= 0.65. This is calibrated for a large collection such as NYPL's. For a smaller collection you may need to set `minimum_featured_quality` to a smaller number to ensure a good diversity of books.

## Integrations

This section is where you connect the Library Simplified components to other services. Some are local pieces of software (Elasticsearch and Postgres), some are infrastructural (CDN, S3), some are sources of books (3M, Overdrive, Axis 360), some are third-party sources of data (Content Cafe, the New York Times), and some connect to other parts of the library (Millenium, Staff Picks).

| Integration name                   | circ | metadata | content |
| ---------------------------------- | ---- | -------- | ------- |
| 3M                                 | O    | X        |         |
| Adobe Vendor ID                    | O    |          |         |
| Axis 360                           | O    |          |         |
| CDN                                | O    |          |         |
| Circulation Manager                | X    |          |         |
| Content Cafe                       |      | O        |         |
| Content Server                     | X    | X        | X       |
| Elasticsearch                      | X    |          |         |
| Google Analytics                   | X    |          |         |
| Google OAuth                       | O    |          |         |
| Metadata Wrangler                  | X    | X        |         |
| Millenium                          | O    |          |         |
| New York Times                     | O    | X        |         |
| Overdrive                          | O    | X        |         |
| Postgres                           | X    | X        | X       |
| S3                                 |      | X        | X       |
| Staff Picks                        | O    |          |         |

### General format

The `integrations` section is a JSON object whose keys are the names
of integrations and whose values are JSON objects. Each JSON object
includes the configuration for one particular integration.

Example:

```
{
    ...
    "integrations" : {
        "Integration #1" : {
	    "url": "http://example.com/"
        },
        "Integration #2" : {
	    "api_key": "foobar123"
        },
	...
    }
    ...
 } 
}
```

### `Content Server`, `Metadata Wrangler`, `Circulation Manager`

These sections control the integration points between the Library
Simplified components themselves. Each includes a single configuration
variable, `url`. This is the URL of the web application. 
When the circulation manager needs to get information from the
metadata wrangler, it looks at the `url` of the `Metadata Wrangler`

Example:

```
...
"Content Server" : {
    "url": "https://oacontent.librarysimplified.org/"
},
...
```

Note that each component needs to know its own `url`. This means that
the circulation manager must itself have a `Circulation Manager`
integration which includes the endpoint URL of the web application.

### 3M

This controls access to the 3M Cloud Library API. The circulation
manager uses it to track a library's 3M collection. The metadata
wrangler uses it to get metadata for books offered by 3M Cloud
Library.

The following keys are all required, and 3M should give you all of
them when you request access to the Cloud Library API:

* `library_id`
* `account_id`
* `account_key`

These keys are standard for book providers:

* `default_loan_period`: The default number of days that a loan will last.
* `default_reservation_period`: The number of days between the time a book becomes available to a patron and the time that book goes to the next patron in the hold queue.

### Adobe Vendor ID

If the circulation manager is supposed to implement the Adobe Vendor
ID service, this integration configures how Adobe expects it to
act. The following keys are all required, and Adobe should give you
all of them once you purchase a Vendor ID license.

* `vendor_id`
* `node_value`

### Axis 360

This controls access to the Axis 360 API. The circulation manager uses
it to track a library's Axis 360 collection. The metadata wrangler
cannot use this, because the Axis 360 API does not let a client get
metadata for books not in the collection.

The following keys are all required, and Baker & Taylor should give you all of
them when you request access to the Axis 360 API:

* `username`
* `password`
* `library_id`
* `server` - this is either "qa" or "production", depending on what Baker & Taylor says

These keys are standard for book providers:

* `default_loan_period`: The default number of days that a loan will last.
* `default_reservation_period`: The number of days between the time a book becomes available to a patron and the time that book goes to the next patron in the hold queue.

### CDN

You can improve your circulation manager's performance by serving book
covers and OPDS feeds through a CDN.

* `book_covers` : Book covers will be served through this CDN host. Attach this CDN host to the S3 bucket that contains your book covers. (You may use NYPL's book covers for this purpose.)

* `opds`: OPDS feeds will be served through this CDN. Attach this CDN host to the web application for your circulation manager.

Example:

```
...
	"CDN" : {
	    "book_covers" : "https://foo.cloudfront.net/",
	    "opds" : "https://bar.cloudfront.net/"
	},
...
```


### Content Cafe

This section configures the metadata wrangler's access to the Content
Cafe API. The metadata wrangler uses this to find book covers,
descriptions, excerpts, and popularity information for works
identified by an ISBN. (This includes all books provided by Axis 360.)
Without this integration, the metadata wrangler cannot find any
information about a book identified solely by an ISBN.

Both of these values are required and are provided by Content Cafe
when you get access to the API.

* `user_id`
* `password`

### Elasticsearch

The circulation manager requires an Elasticsearch server to fulfill
patron searches. Both values are required.

* `url`: The URL to the Elasticsearch server.
* `works_index`: The name of the Elasticsearch index to use. Suggested value: `"circulation-works"`. If this index does not exist it will be automatically created.

### Google Analytics
The circulation manager can optionally send circulation-related events
to your Google Analytics account. To use this integration, set up a
property in Google Analytics. On the administration page for the property,
go to Custom Definitions > Custom Dimensions, and add the following
dimensions (they must be added in this order):
* time
* identifier
* identifier_type
* title
* author
* fiction
* audience
* target_age
* publisher
* language
* genre
* open_access

Each dimension should have the scope set to 'Hit' and the 'Active'
box checked.

Then go to Tracking Info and get the tracking id for the property.
`tracking_id` is the only setting for this integration.

### Google OAuth

The circulation manager uses Google OAuth to validate administrators
for its admin interface. If you haven't set `include_admin_interface`
to `"true"` in the configuration, this integration is not required and
will be ignored.

Use the [Google APIs console](https://console.developers.google.com/)
to create credentials for your application. For local development,
you'll want to add `http://localhost:6500/admin/GoogleAuth/callback`
as an authorized redirect URI; for other environments, the
`/admin/GoogleAuth/callback` route can be added to any configured url.
Once finished, use the "Download JSON" button at the top left of the
Credentials pane to get the JSON required for configuration. The downloaded
file should contain a JSON object with a `web` section including the keys
`client_id`, `project_id`, `token_uri`, `auth_provider_x509_cert_url`, 
`client_secret`, and `redirect_uris`. Paste this JSON object 
into the config file under "Google OAuth".

### Millenium

If access to your collection is managed through Millenium, this
section connects the circulation manager to the Millenium Patron
API. Be sure to also set `"authentication": "Millenium"` in the
`policies` section.

* `url`: The URL to the Millenium Patron API. Example: "http://foo.iii.com:4500/PATRONAPI/".
* `test_username`: A username known to be valid. This is used to verify that Millenium is answering requests correctly.
* `test_password`: The password that goes along with `test_username`.

### New York Times

If you want your circulation manager to have separate lanes for the subset of your collection that was recently on the New York Times bestseller list, you need to [[get an API key|http://developer.nytimes.com/docs/best_sellers_api]] and add it to this integration.

* `best_sellers_api_key`

A metadata wrangler always requires this integration, so that it can service circulation managers that want it.

### Overdrive

This controls access to the Overdrive Library API. The circulation
manager uses it to track a library's Overdrive collection. The metadata
wrangler uses it to get metadata for books offered by Overdrive.

The following keys are all required, and should all be provided you by
Overdrive when you [[request access|https://developer.overdrive.com/application]] to the Overdrive API:

* `client_key`
* `client_secret`
* `website_id`
* `library_id`
* `ils_name`, added in Circulation Manager 2.x . Previous versions of the Circulation Manager relied on using `default` as the value which works for some libraries.

You can also define `cname_url`, if provided by Overdrive, but is currently not used.

These keys are standard for book providers:

* `default_loan_period`: The default number of days that a loan will last.
* `default_reservation_period`: The number of days between the time a book becomes available to a patron and the time that book goes to the next patron in the hold queue.

### Postgres

This is the database configuration. Each Library Simplified component needs its own individual idatabase. The database integration defines the following two variables:

* `production_url`: The database to use in production. Required.
* `test_url`: The database to use when running unit tests. This database will be erased and rebuild on every test run. Optional.

### S3

Amazon S3 is used to host books and book covers. The open access
content server uploads books and generated book covers to S3. The
metadata wrangler uploads scaled-down versions of book covers from
commercial sources.

* `access_key`: Provided by Amazon.
* `secret_key`: Provided by Amazon.
* `book_covers_bucket`: The name of, or the URL to, the S3 bucket that
  contains book covers.
* `open_access_content_bucket`: The name of, or the URL to, the S3 bucket
  that contains open-access content.

All of these fields are required. Your content server and your
metadata wrangler should be configured to use the same buckets.

### Staff Picks

If you want a 'Staff Picks' lane in your circulation manager, you can
have library staff put their picks into a spreadsheet, export the
spreadsheet to CSV, and import it into Library Simplified.

* `type`: Required. Currently the only supported `type` is "csv".
* `url`: The URL to the document containing the staff picks.

## Miscellaneous

| Variable name                      | circ | metadata | content |
| ---------------------------------- | ---- | -------- | ------- |
| data_directory                     |      | X        | X       |
| default_notification_email_address | X    |          |         |
| gutenberg_illustrated_binary_path  |      |          | X       |
| include_admin_interface            | O    |          |         |
| logging                            | O    | O        | O       |

### `data_directory`

This is the directory used to store data that can't be kept in the database for any reason.

On the metadata wrangler, the `Simplified-data` repository should be checked out into this directory. On the content server, this directory should be located on a disk with at least 500 GB of disk space.

### `default_notification_email_address`

Some ebook providers require the patron's email address when putting a book on hold. An email will be sent to that address when the book becomes available. Library Simplified does not gather patrons' email addresses and will do notifications using its own system (not implemented yet). This value should be an email address that will always bounce. Once the notification system is implemented, this value should be the email endpoint to the notification system.


### `include_admin_interface`

If you would like to launch the application with an admin interface, you'll need to set this property to `true`. The admin interface also requires the policy `admin_authentication_domain` to be set and the `Google OAuth` integration to be configured.

### `logging`

This is a JSON object that determines how logs are generated.

* `level`: The logging level. The value is one of the standard log levels: `"DEBUG"`, `"INFO"`, and so on.
* `output`: The output type. The value is either `"text"` or `"json"`.