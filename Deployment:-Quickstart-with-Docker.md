So you're deploying your library's circulation manager. Awesome! If you'd like to get up and running quickly, we recommend using our Docker image.

If you're already familiar with Docker and/or would like to contribute to our Docker builds, you can find our build files at [NYPL-Simplified/circulation-docker](https://github.com/NYPL-Simplified/circulation-docker).

#### Circulation Manager

##### *Local Prep*

1. **Create your configuration file.** On your local machine, use [this documentation](Configuration) to create the JSON file for your particular library's configuration. If you're unfamiliar with Json, you can use [this JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/#) to validate your configuration file.

2. Name your file `config.json` and **put it on your production server** at `/etc/libsimple`. (You can put the file in any directory you'd like, but you'll need to change the value in the commands below accordingly.) For the rest of the instructions, we'll be working on this server.

##### *On the Host Server*

1. **Install Docker.** Docker has [step-by-step instructions](https://docs.docker.com/engine/installation/linux/) to grab its most up-to-date version. Depending on your package manager, you could also install a slightly older version with: `sudo apt-get install docker-engine` or `sudo yum install docker-engine`.

2. **Get the Docker images** for the Library Simplified Circulation Manager. Run:

    ```sh
    $ sudo docker pull nypl/circ-deploy && sudo docker pull nypl/circ-scripts
    ```

3. **Create any dependent, temporary containers** (optional) for integrations like Elasticsearch and Postgres. *We don't recommend using containers in the long-term for holding or maintaining data.* However, if you just want to get a sense of how your Circulation Manager will work, containers are a quick option. Instructions for integrating [Elasticsearch](#es) and [Postgres](#pg) via Docker can be found below.

4. **Create a Circulation Manager script-running container.** Now we need to fill in that empty OPDS feed with your library's books, which will require running a number of scripts. Read the details below about the arguments you're passing before running this script; you will probably need to alter it to meet your needs.

    ```sh
    $ sudo docker run -d --name circ-scripts \
        -e TZ="US/Central" \
        # set the variable below if you have never used the configured database before
        -e LIBSIMPLE_DB_INIT="true" \
        -v /etc/libsimple:/etc/circulation \
        nypl/circ-scripts
    ```

    *What you're doing.* You're running this container in detached mode (`-d`), passing in your configuration file to where it needs to be (`-v`), and calling it "circ-scripts". With the (`-e`) optional argument `TZ`, you can pass a [Debian-system timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) representing your local time zone, which will cause timed scripts to run according to your local time. If the database you've connected in your configuration has never been used before, use (`-e`) to set the optional argument `LIBSIMPLE_DB_INIT`. This will keep track of the state of the database you've created and create an alias on your Elasticsearch cluster, allowing database updates to be easily managed with scripts.

    *Troubleshooting.* You'll want to check the logs of your container. For example:

    ```sh
    # check logs of running supervisor processes
    $ sudo docker logs circ-scripts

    # check logs of cron and scripts
    $ sudo docker exec circ-scripts cat /var/log/cron.log | less
    $ sudo docker exec circ-scripts ls /var/log/libsimple
    $ sudo docker exec circ-scripts cat /var/log/libsimple/overdrive_monitor_full | less

    # The log directory can also be found on the production server.
    # Its location can be found using this command.
    $ sudo docker inspect circ-scripts \
      --format='{{range $mount := .Mounts}}{{if eq $mount.Destination "/etc/circulation"}}{{$mount.Source}}{{end}}{{end}}'
    ```

5. **Create a Circulation Manager deployment container.** If you are creating these containers for the first time, only run the deployment container **AFTER** you've created the scripts container, or you run the risk of generating [the IntegrityErrors described in #20](https://github.com/NYPL-Simplified/circulation-docker/issues/20). Should you face this foul beast, run `sudo docker exec circ-deploy touch uwsgi.ini` to reload the application without error.

    ```sh
    $ sudo docker run -d -p 80:80 --name circ-deploy \
        -v /etc/libsimple:/etc/circulation \
        # only set the below variable if you haven't created a scripts container or otherwise used the configured db before
        -e LIBSIMPLE_DB_INIT="true" \
        nypl/circ-deploy
    ```

    *What you're doing.* You're running this container in detached mode (`-d`), binding its port 80 to your server's port 80 (`-p`), passing in your configuration file where it needs to be (`-v`) and calling it "circ-deploy". When you visit your server through a browser, you'll see a very sparse OPDS feed. If the database you've connected in your configuration has never been used before, use (`-e`) to set the optional argument `LIBSIMPLE_DB_INIT`. This will keep track of the state of the database you've created and create an alias on your Elasticsearch cluster, allowing database updates to be easily managed with scripts.

    *Troubleshooting.* You'll want to check the logs of your container (`/var/log/nginx/error.log` and `/var/www/circulation/uwsgi.log`) to troubleshoot:

    ```sh
    # check logs of running supervisor processes
    $ sudo docker logs circ-deploy

    # check logs inside the container
    $ sudo docker exec circ-deploy cat /var/log/nginx/error.log | less
    $ sudo docker exec circ-deploy cat /var/www/circulation/uwsgi.log | less
    ```

6. **Confirm your scripts are running.** Once you've given your scripts some time to run (~30 minutes should be enough time to start having works move through the import process), you'll want to refresh your views so they show up in your deployed app.

    ```sh
    $ sudo docker exec circ-scripts /var/www/circulation/core/bin/run refresh_materialized_views
    ```

7. **Maintenance.** You can hop into a running container at any time with the command:
    ```sh
    $ sudo docker exec -it circ /bin/bash
    ```

    Docker has fantastic documentation to get more familiar with its command line tools, like `docker exec` and `docker inspect`. We recommend you [check them out](https://docs.docker.com/engine/reference/commandline/cli/).


##### *Evaluating Success*

If your Docker containers are running successfully, you should have a `/var/log/libsimple` directory full of logfiles in your circ-scripts container, and you should be able to visit your server's domain and see an OPDS feed from circ-deploy. If either of these things aren't occurring, use the troubleshooting details above to check `var/log/cron.log` or the logfiles in `/var/log/libsimple` for circ-scripts and/or `/var/www/circulation/uwsgi.log` or `/var/log/nginx/error.log`.


#### <a name='es'></a>*Elasticsearch* (optional support container)

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
    $ sudo docker inspect es --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}"'
    ```

3. **Add the Elasticsearch URL to your configuration file.** When you run `sudo docker ps`, you'll see a single running container called es. Use the IP that comes from running `inspect` to update your your `config.json` file with the proper Elasticsearch location. You should end up with something like `"http://172.17.0.2:9200"`.


#### <a name='pg'></a>*Postgres* (optional support container)

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
    $ sudo docker inspect pg --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}"'
    ```

3. **Create a Postgres database.** Run:

    ```sh
    $ docker exec -u postgres pg psql -c "create user simplified with password 'test';"    # create a user and password
    $ docker exec -u postgres pg psql -c "create database simplified_circ_db;"            # create database
    $ docker exec -u postgres pg psql -c "grant all privileges on database simplified_circ_db to simplified;"
    ```

4. **Add the Postgres URL to your configuration file.** In `config.json`, add the appropriate  `production_url`. You should end up with something like `"postgres://simplified:test@172.17.0.3:5432/simplified_circ_db"`, following the `"postgres://<USERNAME>:<PASSWORD>@<HOST>:<PORT>/<DATABASE_NAME>"` format.