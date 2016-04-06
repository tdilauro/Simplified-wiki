So you're deploying your library's circulation manager. Awesome! If you'd like to get up and running quickly, we recommend using our Docker image.

##### *Local Prep*

1. **Create your configuration file.** On your local machine, use [this documentation](Configuration) to create the JSON file for your particular library's configuration. If you're unfamiliar with Json, you can use [this JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/#) to validate your configuration file.

2. Name your file `config.json` and **put it on your production server** at `/var/www/config.json`. For the rest of the instructions, we'll be working on this server.

##### *On the Host Server*

1. **Install Docker.** Docker has [step-by-step instructions](https://docs.docker.com/linux/step_one/) to grab its most up-to-date version. Depending on your package manager, you could also install a slightly older version with: `sudo apt-get install docker.io` or `sudo yum install docker.io`.

2. **Get the Docker image** for both Elasticsearch v1.x and the Library Simplified Circulation Manager. Run:

    ```sh
    $ sudo docker pull elasticsearch:1 && sudo docker pull nypl/circ-deploy && sudo docker pull nypl/circ-scripts
    ```

3. **Create an Elasticsearch container,** and grab its IP Address. Run:

    ```sh
    $ sudo docker run -d --name es elasticsearch:1     # create an elasticsearch container
    $ sudo docker ps                                   # confirm that it's running
    $ sudo docker inspect es | grep 'IPAddress'        # note its IP address
    ```

   When you run `sudo docker ps`, you'll see a single running container called es. Use the IP that comes from running `inspect` to update your your `config.json` file with the proper Elasticsearch location. You should end up with something like `"http://172.17.0.2:9200"`.

4. **Create a Circulation Manager deployment container.**

    ```sh
    $ sudo docker run -d -p 80:80 --name circ-deploy \
        -v /var/www/config.json:/var/www/circulation/config.json \
        nypl/circ-deploy
    ```

    *What you're doing.* You're running this container in detached mode (`-d`), binding its port 80 to your server's port 80 (`-p`), passing in your configuration file where it needs to be (`-v`) and calling it "circ-deploy". When you visit your server through a browser, you'll see a very sparse OPDS feed.

    *Troubleshooting.* You'll want to check the logs of your container (`/var/log/nginx/error.log` and `/var/www/circulation/uwsgi.log`) to troubleshoot:

    ```sh
    # check logs of running supervisor processes
    $ sudo docker logs circ-deploy

    # check logs inside the container
    $ sudo docker exec circ-deploy cat /var/log/nginx/error.log | less
    $ sudo docker exec circ-deploy cat /var/www/circulation/uwsgi.log | less
    ```

5. **Create a Circulation Manager script-running container.** Now we need to fill in that empty OPDS feed with your library's books, which will require running a number of scripts. Read the details below about the arguments you're passing before running this script; you will probably need to alter it to meet your needs.

    ```sh
    # Depending on your needs, it could also be as simple as running:
    $ sudo docker run -d -v /var/www/config.json:/var/www/circulation/config.json \
        --name circ-scripts nypl/circ-scripts

    # However, if you're running scripts for the first time ever and your library resides in the Central timezone,
    # has a ThreeM account, and began business with ThreeM on April 1, 2014:
    $ sudo docker run -d --name circ-scripts \
        -e timezone="US/Central" \
        -e libsimple_init=true \
        -e threem_start_date="2014-04-01" \
        -v /var/www/config.json:/var/www/circulation/config.json \
        nypl/circ-scripts
    ```

    *What you're doing.* You're running this container in detached mode (`-d`), passing in your configuration file to where it needs to be (`-v`), and calling it "circ-scripts". Each of the (`-e`) options passes in an optional variable that you may or may not need, depending on when and where you're creating the container.

    | Argument | Value | Default | Purpose |
    | --- | --- | --- | --- |
    | timezone | a [Debian-system timezone](http://manpages.ubuntu.com/manpages/saucy/man3/DateTime::TimeZone::Catalog.3pm.html) representing your local time | "US/Eastern" | Allows timed scripts to run according to your local time |
    | libsimple_init | true | null | Runs scripts that must be specifically initialized. (Currently this only impacts ThreeM accounts.) |
    | threem_start_date | a date, formatted YYYY-MM-DD | two years before the day you create the container | Determines at what point we should start looking for active ThreeM licenses. |

    *Troubleshooting.* You'll want to check the logs of your container. For example:

    ```sh
    # check logs of running supervisor processes
    $ sudo docker logs circ-scripts

    # check logs of cron and scripts
    $ sudo docker exec circ-scripts cat /var/log/cron.log | less
    $ sudo docker exec circ-scripts ls /var/log/libsimple
    $ sudo docker exec circ-scripts cat /var/log/libsimple/overdrive_monitor_full | less
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
