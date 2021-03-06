These directions come primarily from [a very helpful tutorial created by Vladik Khononov](http://vladikk.com/2013/09/12/serving-flask-with-nginx-on-ubuntu/).

1. Install nginx.

    ```sh
    # This package will keep nginx up to date.
    $ sudo yum install epel-release
    $ sudo yum install nginx
    
    # Confirmation step:
    $ sudo service nginx start
    ```
    When you navigate to your server's public address in a browser, you should see a generic "Welcome to nginx" page.

2. Install uwsgi in your Library Simplified repo's virtual environment.

    ```sh
    # If you're not already in your LS repo, go there, e.g.
    $ cd YOUR_LS_APP_DIR
    
    # Pop into your Python environment & install uWSGI
    $ source env/bin/activate
    $ pip install uwsgi
    ```

3. Create an appropriately-named directory in a location where nginx will have access. We'll refer to it in the next step as YOUR_UWSGI_SOCKET_DIR. Make sure that both nginx and ec2-user has permissions.

    ```
    $ mkdir /var/www/circulation
    $ sudo chown nginx:nginx /var/www/circulation
    $ sudo chmod 777 /var/www/circulation
    ```

4. In your LS repo, configure nginx with an `nginx.conf` file. **For the most up-to-date example, use the file found [here](https://github.com/NYPL-Simplified/circulation-docker/blob/master/deploy/nginx.conf).** Be sure to change the `uwsgi_pass` variable to reference `unix:YOUR_UWSGI_SOCKET_DIR/uwsgi.sock`.

5. Then create a symbolic link to it and restart nginx. 

    ```sh
    $ sudo ln -s /var/www/circulation/nginx.conf /etc/nginx/conf.d/circulation.conf
    $ sudo service nginx restart
    ```
    Now when you navigate to your server's address in a web browser, you should get a 502 error. Fantastic!

6. In your LS repo, configure uWSGI with an `uwsgi.ini` file. **For the most up-to-date example, use the file found [here](https://github.com/NYPL-Simplified/circulation-docker/blob/master/deploy/uwsgi.ini).** Be sure to set the `base` variable to YOUR_LS_APP_DIR, and the `socket` variable to `YOUR_UWSGI_SOCKET_DIR/%n.sock`

7. Enter the virtual environment, and confirm that your uWSGI configuration works.

    ```
    $ cd YOUR_LS_APP_DIR
    $ source env/bin/activate
    $ uwsgi --ini uwsgi.ini
    ```

    You'll see a single line of text in your terminal: `[uWSGI] getting INI configuration from uwsgi.ini`, followed by a blank link. Before ending the process, visit your server's address in the web browser, and you should an OPDS feed that represents your LS app! Make sure that if you're configuring the Metadata Wrangler, you try the `/lookup` route for a feed, as the home index is not currently being used.

8. If you experience errors during step #7, look to `/var/log/nginx/errors.log` and `/var/log/uwsgi/emperor.log` for information about what's going wrong.

9. Once things are working, run uWSGI in the background:
    ```sh
    $ source env/bin/activate
    $ uwsgi --ini uwsgi.ini &
    ```