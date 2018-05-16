All configuration of the Library Simplified circulation manager happens through the administrative interface. Configuration details are stored in a database. But which database?

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

There is no longer any need to use a JSON configuration file, so the instructions on how that file works have been removed to avoid confusion.