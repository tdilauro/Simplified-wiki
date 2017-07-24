So you're deploying your library's circulation manager. Awesome! If you'd like to get up and running quickly, we recommend using our Docker image.

If you're already familiar with Docker and/or would like to contribute to our Docker builds, you can find our build files at [NYPL-Simplified/circulation-docker](https://github.com/NYPL-Simplified/circulation-docker).

## Contents:
- Running the Circulation Manager
  - [Prep work](#cm-prep)
  - [Creating Circulation Manager containers](#cm-host)
    - [Running Scripts](#cm-scripts)
    - [Deploying the App](#cm-app)
    - [Environment Variables](#cm-env)
    - [Evaluating Success](#cm-success)
- Support Containers for Testing and Development
  - [Creating a Postgres container](#pg)
  - [Creating an Elasticsearch container](#es)

---

## <a name='cm-prep'></a>Prep Work

1. **Create your configuration file.**

    1. On your local machine, use [this documentation](Configuration) to create the JSON file for your particular library's configuration. If you're unfamiliar with Json, you can use [this JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/#) to validate your configuration file.
    2. Name your file `config.json` and **put it on your production server** at `/etc/libsimple`. (You can put the file in any directory you'd like, but you'll need to change the value in the commands below accordingly.) For the rest of the instructions, we'll be working on this server.

2. **Install Docker.** Docker has [step-by-step instructions](https://docs.docker.com/engine/installation/linux/) to grab its most up-to-date version. Depending on your package manager, you could also install a slightly older version with: `sudo apt-get install docker-ce` or `sudo yum install docker-ce`.

3. **Create any dependent, temporary containers** (optional) for integrations like Elasticsearch and Postgres. *We don't recommend using containers in the long-term for holding or maintaining data.* However, if you just want to get a sense of how your Circulation Manager will work, containers are a quick option. Instructions for integrating [Elasticsearch](#es) and [Postgres](#pg) via Docker can be found below.

4. **Get the Docker images** for the Library Simplified Circulation Manager. Run:

    ```sh
    $ sudo docker pull nypl/circ-deploy && sudo docker pull nypl/circ-scripts
    ```

## <a name='cm-host'></a>Running Circulation Manager containers

### <a name='cm-scripts'></a>Running scripts

To deploy an app filled with your library's books, you'll need to run a number of scripts. Read [the environment variable details below about](#cm-env) before running this script; you will likely need to alter it to meet your needs.

#### Example `docker run` script

```sh
$ sudo docker run -d --name circ-scripts \
    -e TZ="US/Central" \
    -v /etc/libsimple:/etc/circulation \
    -e SIMPLIFIED_CONFIGURATION_FILE='/etc/circulation/config.json' \
    -e SIMPLIFIED_DB_TASK='init' \
    nypl/circ-scripts
```

#### What It Does

The example above runs this resulting container in detached mode (`-d`), passing in your prepared configuration file to where it needs to be (`-v`, `-e SIMPLIFIED_CONFIGURATION_FILE`) and calling it "circ-scripts". With the (`-e`) optional argument `TZ`, you can pass a [Debian-system timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) representing your local time zone, which will cause timed scripts to run according to your local time. If the database you've connected in your configuration has never been used before, use `-e` to set the optional argument `SIMPLIFIED_DB_TASK` to `'init'`. This will keep track of the state of the database you've created and create an alias on your Elasticsearch cluster, allowing database updates to be easily managed with scripts.

#### Running Scripts

Once you've given your scripts some time to run (~30 minutes should be enough time to start having works move through the import process), you'll want to refresh your views so they show up in your deployed app.

    ```sh
    $ sudo docker exec circ-scripts /var/www/circulation/core/bin/run refresh_materialized_views
    ```

#### Troubleshooting

You'll want to check the logs of your container. For example:

```sh
# check logs of the database task and running supervisor processes
$ sudo docker logs circ-scripts

# check logs of cron and scripts
$ sudo docker exec circ-scripts cat /var/log/cron.log | less
$ sudo docker exec circ-scripts ls /var/log/libsimple
$ sudo docker exec circ-scripts cat /var/log/libsimple/overdrive_monitor_full | less

# The log directory can also be found on the production server.
# Its location can be found using this command.
$ sudo docker inspect circ-scripts \
  --format='{{range $mount := .Mounts}}{{if eq $mount.Destination "/var/log"}}{{$mount.Source}}{{end}}{{end}}'
```

You can hop into a running container at any time with the command: `$ sudo docker exec -it circ /bin/bash`

Docker has fantastic documentation to get more familiar with its command line tools, like `docker exec` and `docker inspect`. We recommend you [check them out](https://docs.docker.com/engine/reference/commandline/cli/).

### <a name='cm-app'></a>Deploying the App

Using an `nypl/circ-deploy` container deploys the OPDS feeds expected by the SimplyE client applications. Read [the environment variable details below about](#cm-env) before running the following script; you will likely need to alter it to meet your needs.

#### Example `docker run` script

```sh
$ sudo docker run -d -p 80:80 --name circ-deploy \
    -v /etc/libsimple:/etc/circulation \
    -e SIMPLIFIED_CONFIGURATION_FILE='/etc/circulation/config.json' \
    -e SIMPLIFIED_DB_TASK="migrate" \
    nypl/circ-deploy
```

#### What It Does

The script above runs the container in detached mode (`-d`), binding its port 80 to your server's port 80 (`-p`), passing in your configuration file where it needs to be (`-v`, `-e SIMPLIFIED_CONFIGURATION_FILE`) and calling it "circ-deploy". Unless you've been running a scripts container for while, when you visit your server through a browser, you'll see a very sparse OPDS feed. If the database you've connected in your configuration has never been used before, use `-e` to set the optional argument `SIMPLIFIED_DB_TASK` to `'init'`. This will keep track of the state of the database you've created and create an alias on your Elasticsearch cluster, allowing database updates to be easily managed with scripts.

#### Troubleshooting

You'll want to check the logs of your container (`/var/log/nginx/error.log` and `/var/log/libsimple/uwsgi.log`) to troubleshoot:

```sh
# check logs of the database task and running supervisor processes
$ sudo docker logs circ-deploy

# check logs inside the container
$ sudo docker exec circ-deploy cat /var/log/nginx/error.log | less
$ sudo docker exec circ-deploy cat /var/log/libsimple/uwsgi.log | less

# restart the application
$ sudo docker exec circ-deploy touch uwsgi.ini
```

You can hop into a running container at any time with the command: `$ sudo docker exec -it circ /bin/bash`

Docker has fantastic documentation to get more familiar with its command line tools, like `docker exec` and `docker inspect`. We recommend you [check them out](https://docs.docker.com/engine/reference/commandline/cli/).

### <a name='cm-env'></a>*Environment Variables*

#### `SIMPLIFIED_CONFIGURATION_FILE`

*Required in v1.1 only. Optional in v2.x.* The full path to configuration file in the container. Using the volume `-v` for v1.1, it should look something like `/etc/circulation/YOUR_CONFIGURATION_FILENAME.json`. In v2.x you can volume it in wherever you'd like.

Use [this documentation](https://github.com/NYPL-Simplified/Simplified/wiki/Configuration) to create the JSON file for your particular library's configuration. If you're unfamiliar with JSON, you can use [this JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/#) to validate your configuration file.

#### `SIMPLIFIED_DB_TASK`

*Required.* Performs a task against the database at container runtime. Options are:
  - `ignore` : Does nothing. This is the default value.
  - `init` : Initializes the app against a brand new database. If you are running a circulation manager for the first time every, use this value to set up an Elasticsearch alias and account for the database schema for future migrations.
  - `migrate` : Migrates an existing database against a new release. Use this value when switching from one stable version to another.

#### `SIMPLIFIED_PRODUCTION_DATABASE`

*Required in v2.x only.* The URL of the production PostgreSQL database for the application.

#### `SIMPLIFIED_TEST_DATABASE`

*Optional in v2.x only.* The URL of a PostgreSQL database for tests. This optional variable allows unit tests to be run in the container.

#### `TZ`

*Optional. Scripts container only.* The timezone of the library or libraries on this circulation manager, selected according to [Debian-system timezone options](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). This value allows scripts to run at ideal times.

### <a name='cm-success'></a>*Evaluating Success*

If your Docker containers are running successfully, you should have a `/var/log/libsimple` directory full of logfiles in your circ-scripts container, and you should be able to visit your server's domain and see an OPDS feed from circ-deploy. If either of these things aren't occurring, use the troubleshooting details above to check `var/log/cron.log` or the logfiles in `/var/log/libsimple` for circ-scripts and/or `/var/log/libsimple/uwsgi.log` or `/var/log/nginx/error.log`.

## Support Containers (for use in development or testing)
### <a name='es'></a>*Elasticsearch*

While we do **not** recommend you run Elasticsearch from a Docker container permanently, you may want to get up and running with a throwaway search index. Elasticsearch isn't installed via the Dockerfile, so the fastest way to connect to it will be through another container. Here's how:

1. **Get the Docker image** for Elasticsearch v1.x:

    ```sh
    $ sudo docker pull elasticsearch:1
    ```

2. **Create an Elasticsearch container,** and grab its IP Address. Run:

    ```sh
    $ sudo docker run -d --name es elasticsearch:1     # create an elasticsearch container
    $ sudo docker ps                                   # confirm that it's running
    # note its IP address
    $ sudo docker inspect es --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}"
    ```

3. **Add the Elasticsearch URL to your configuration file.** When you run `sudo docker ps`, you'll see a single running container called es. Use the IP that comes from running `inspect` to update your your `config.json` file with the proper Elasticsearch location. You should end up with something like `"http://172.17.0.2:9200"`.


### <a name='pg'></a>*Postgres*

While we do **not** recommend you run Postgres from a Docker container permanently, you may want to get up and running with a throwaway database. Postgres isn't installed via the Dockerfile, so the best way to connect to Postgres will be through another container. Here's how:

1. **Get the Docker image** for Postgres 9.4 or 9.5:

    ```sh
    $ sudo docker pull postgres:9.5
    ```

2. **Create a Postgres container,** and grab its IP Address. Run:

    ```sh
    $ sudo docker run -d --name pg postgres:9.5        # create a postgres container
    $ sudo docker ps                                   # confirm that it's running
    # note its IP address
    $ sudo docker inspect pg --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}"
    ```

3. **Create a Postgres database.** Run:

    ```sh
    $ docker exec -u postgres pg psql -c "create user simplified with password 'test';"    # create a user and password
    $ docker exec -u postgres pg psql -c "create database simplified_circ_db;"            # create database
    $ docker exec -u postgres pg psql -c "grant all privileges on database simplified_circ_db to simplified;"
    $ docker exec -u postgres pg psql -d simplified_circ_db -c "create extension pgcrypto;"
    ```

4. **Add the Postgres URL to your configuration file.** In `config.json`, add the appropriate  `production_url`. You should end up with something like `"postgres://simplified:test@172.17.0.3:5432/simplified_circ_db"`, following the `"postgres://<USERNAME>:<PASSWORD>@<HOST>:<PORT>/<DATABASE_NAME>"` format.