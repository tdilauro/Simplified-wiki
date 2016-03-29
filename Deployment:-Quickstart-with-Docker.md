So you're deploying your library's circulation manager. Awesome! If you'd like to get up and running quickly, we recommend using our Docker image.

##### *Local Prep*

1. **Create your configuration file.** On your local machine, use [this documentation](Configuration) to create the JSON file for your particular library's configuration. If you're unfamiliar with Json, you can use [this JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/#) to validate your configuration file.

2. Name your file `config.json` and **put it on your production server** at `/var/www/config.json`. For the rest of the instructions, we'll be working on this server.

##### *On the Host Server*

1. **Install Docker.** Docker has [step-by-step instructions](https://docs.docker.com/linux/step_one/) to grab its most up-to-date version. Depending on your package manager, you could also install a slightly older version with: `sudo apt-get install docker.io` or `sudo yum install docker.io`.

2. **Get the Docker image** for both Elasticsearch v1.x and the Library Simplified Circulation Manager. Run:

    ```sh
    $ sudo docker pull elasticsearch:1 && sudo docker pull nypl/circulation
    ```

3. **Create an Elasticsearch container,** and grab its IP Address. Run:

    ```sh
    $ sudo docker run -d --name es elasticsearch:1     # create an elasticsearch container
    $ sudo docker ps                                   # confirm that it's running
    $ sudo docker inspect es | grep 'IPAddress'        # note its IP address
    ```

   When you run `sudo docker ps`, you'll see a single running container called es. Use the IP that comes from running `inspect` to update your your `config.json` file with the proper Elasticsearch location. You should end up with something like `http://172.17.0.2:9200`

4. **Create a Circulation Manager container.** Running the following command will plop you directly into the shell for your container, as root.

    ```sh
    $ sudo docker run -it -p 80:80 -v /var/www/config.json:/var/www/circulation/config.json --name circ nypl/circulation
    ```

    *What you're doing.* You're running this container in interactive mode (`-it`), binding its port 80 to your server's port 80 (`-p`), passing in your configuration file where it needs to be (`-v`) and calling it "circ".

5. **Start the necessary services in the container.** You need to start Nginx and UWSGI to 

    ```sh
    # From inside your 'circ' Docker container
    $ service nginx start
    $ source env/bin/activate
    $ uwsgi --ini uwsgi.ini &
    ```

    When you visit your IP address now, you should see an OPDS feed. If you don't, you'll want to check the logs of your container to troubleshoot: `/var/log/nginx/error.log` and `/var/www/circulation/uwsgi.log`.

6. **Do _not_ exit out of the container session with `exit`** or your container (and thus your deployed site) will end. Instead, use the key binding `CTRL+p CTRL+q` to detach from the container. You can reattach at any time with the command `sudo docker attach circ`.