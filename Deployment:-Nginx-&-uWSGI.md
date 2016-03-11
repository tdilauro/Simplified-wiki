These directions come primarily from [a very helpful tutorial created by Vladik Khononov](http://vladikk.com/2013/09/12/serving-flask-with-nginx-on-ubuntu/).

1. Install nginx.

    ```sh
    # This package will keep nginx up to date.
    $ sudo yum install epel-release
    $ sudo yum install nginx
    
    # Confirmation step:
    $ sudo /etc/init.d/nginx start
    ```
    When you navigate to your server's public address in a browser, you should see a generic "Welcome to nginx" page.

2. Install uwsgi in your Library Simplified repo's virtual environment.

    ```sh
    # If you're not already in your LS repo, go there, e.g.
    $ cd $YOUR_LS_APP_DIR
    
    # Pop into your Python environment & install uWSGI
    $ source env/bin/activate
    $ pip install uwsgi
    ```

3. Create an appropriately-named directory in a location where nginx will have access. We'll refer to it in the next step as $YOUR_UWSGI_SOCKET_DIR.

    `$ mkdir /var/www/circulation`

4. In your LS repo, configure nginx with an `nginx.conf` file. For example:

    ```
    server {
        listen      80 deferred;
        server_name $YOUR.SERVER.IP.ADDRESS;
        charset     utf-8;
        client_max_body_size 75M;
    
        location / { try_files $uri @circulation; }
        location @circulation {
            include uwsgi_params;
            uwsgi_read_timeout 120;
            uwsgi_pass unix:$YOUR_UWSGI_SOCKET_DIR;
        }
    }
    ```

5. Then create a symbolic link to it and restart nginx. 

    ```sh
    $ sudo ln -s /var/www/circulation/nginx.conf /etc/nginx/conf.d/circulation.conf
    $ service nginx restart
    ```
    Now when you navigate to your server's address in a web browser, you should get a 502 error. Fantastic!

6. In your LS repo, configure uWSGI with an `uwsgi.ini` file. For example:

    ```
    [uwsgi]
    # full path to the application's base folder
    base = $YOUR_LS_APP_DIR
    
    # python module to import & environment details
    app = app
    module = %(app)
    home = %(base)/env
    pythonpath = %(base)

    # socket file's location & permissions
    socket = $YOUR_UWSGI_SOCKET_DIR
    chmod-socket = 666

    callable = app
    logto = %(base)/%n.log
    touch-reload = uwsgi.ini
    ```

7. Confirm that your uWSGI configuration works.

    `$ uwsgi --ini $YOUR_LS_APP_DIR/uwsgi.ini`

    When you visit your server's address in the web browser, you should see your LS app! Make sure that if you're configuring the Metadata Wrangler, you try the `/lookup` route for a feed, as the home index is not currently being used.

8. Configure uWSGI Emperor with a `/etc/init/uwsgi.conf` file. 

    You may choose another way to push the work of running your app to a background file, but Emperor is an option. Here's what that configuration file might look like:

    ```
    description "uWSGI"
    start on runlevel [2345]
    stop on runlevel [!2345]
    respawn

    env UWSGI=$YOUR_LS_APP_DIR/env/bin/uwsgi
    env LOGTO=/var/log/uwsgi/emperor.log

    script
        source $YOUR_LS_APP_DIR/env/bin/activate
        $UWSGI --master --emperor /etc/uwsgi/vassals --die-on-term --uid ec2-user --gid ec2-user --logto $LOGTO
    end script
    ```

9. Finally, create a vassals directory and link your uwsgi.ini file there:
    ```sh
    $ mkdir /etc/uwsgi/vassals
    $ sudo ln -s uwsgi.ini /etc/uwsgi/vassals
    ```

10. Run `sudo start uwsgi` and make sure that everything's working properly. Look to `/var/log/nginx/errors.log` and `/var/log/uwsgi/emperor.log` for errors as you go.