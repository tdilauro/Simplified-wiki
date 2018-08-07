Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

# Install packaged requirements

## All applications

### On Red Hat systems:

```
sudo yum install python27 python27-devel git postgresql python-nose python-sqlalchemy python-pip nginx
sudo pip install virtualenv virtualenvwrapper

# These are necessary for installing python lxml through pip.
sudo yum install libxml2-devel libxslt-devel gcc libxslt-python

# This is necessary for installing psycopg2 through pip
sudo yum install postgresql-devel

# Used by pillow to generate JPG thumbnails.
sudo yum install libjpeg libjpeg-devel

# Used by cairosvg to generate SVG thumbnails.
sudo yum install libffi-devel pycairo
```

### On Ubuntu systems:

```
sudo apt-get install python-dev git postgresql python-nose python-sqlalchemy python-pip nginx
sudo pip install virtualenv virtualenvwrapper
sudo apt-get install libxml2-dev libxslt-dev gcc python-libxml2

# This is necessary for installing psycopg2 through pip
sudo apt-get install postgresql-server-dev-9.3

# Used by pillow to generate JPG thumbnails.
sudo apt-get install libjpeg-dev

# Used by cairosvg to generate SVG thumbnails.
sudo apt-get install libffi-dev python-cairo

sudo apt-get install python-numpy
```

### On Mac:

1. Install homebrew:
    ```ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"```
2. Install the XCode command line tools when prompted.
3. Download and install Postgres.app from postgresapp.com.
4. Set up SSH keys using github's instructions: https://help.github.com/articles/generating-ssh-keys/
5. Install Python, pip, virtualenv, and Postgres executables

    ```
    # install python 2.7, with pip
    brew install python2
    # install Node and NPM
    brew install node
    # install virtualenv
    pip install virtualenv
    # Add postgres executables to PATH
    export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/9.6/bin/
    ```
6. Install additional dependencies
```
    brew install cairo --use-clang
    brew install py2cairo
    brew install libjpeg
    brew install homebrew/dupes/zlib
```

## Metadata wrangler only
The metadata wrangler has a number of additional dependencies so that scikit-learn can be installed:

```
# Metadata wrangler only
sudo yum install numpy python-matplotlib ipython python-pandas sympy python-nose gcc gcc-c++ lapack-devel blas-static lapack-static
```

Ubuntu:
```
sudo apt-get install liblapack-dev
```

## Set up the Elasticsearch service (Circulation manager only)

Library Simplified is currently dependent on Elasticsearch version 1.x. Later Elasticsearch versions will raise errors.

### Linux

Install the JDK:

```
sudo apt-get install openjdk-7-jre
```

Then, follow the instructions here: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html

Make sure Elasticsearch starts on bootup! Here's how to start it manually:

```
sudo service elasticsearch start
```

### Mac OS X

The circulation manager [currently requires Elasticsearch 1.5-1.7](https://github.com/NYPL-Simplified/server_core/issues/674), which is no longer supported. It's not possible to install it with Homebrew and it doesn't work with the latest version of Java. If possible, it's easier to point to an Elasticsearch instance hosted elsewhere so you don't need to set it up locally.

If you really want to set it up, you'll first need to uninstall any versions of Java more recent than 8. For example, to uninstall Java 9 on OS 10.13, run:
```
sudo rm -rf "/Library/Internet Plug-Ins/JavaAppletPlugin.plugin"
sudo rm -rf "/Library/PreferencePanes/JavaControlPanel.prefPane"
sudo rm -rf /Library/Java/JavaVirtualMachines/jdk-9.jdk/
```

Then install the Java 8 JDK, from [the installer on Oracle's site](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

Finally, download, extract, and run elasticsearch:
```
curl -L -O https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.6.tar.gz
tar -xvf elasticsearch-1.7.6.tar.gz
cd elasticsearch-1.7.6/bin
./elasticsearch
```

Check http://localhost:9200 to make sure it's running.

# Check out the Simplified repositories

You're probably checking out the circulation manager. Here's the command:

```
git clone git@github.com:NYPL-Simplified/circulation.git circulation
cd circulation
```

Make sure you [[have an SSH key associated with your Github account|https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/]], or you won't be able to clone a repository over SSH.

If you're checking out the metadata wrangler, the command is the same but the repository is different.

```
git clone git@github.com:NYPL-Simplified/metadata_wrangler.git metadata
cd metadata
```

# Initialize the submodules

Both product repositories include a Git submodule for the `Simplified-server-core` project, which defines common things like the database schema. You'll need to initialize this module.

```
git submodule init
git submodule update
```

When you run `git submodule update`, the `Simplified-server-core` project will be cloned into the `core` directory.

# Initialize the administrative interface (circulation manager only)

The circulation manager includes a front-end administrative interface written in Node. To get this working, you'll need to link an additional project called `circulation-web` to the circulation manager.

The simplest way to install this project is to run `npm install` within the `circulation/api/admin` directory:

```
cd api/admin
npm install
```

If you plan to do development on the administrative interface, you'll need to clone its repository rather than installing the latest release. Here's how to do that:

```
git clone git@github.com:NYPL-Simplified/circulation-web.git circulation-web
cd circulation-web
npm link
cd ../api/admin
npm link simplified-circulation-web
```

# Set up database users

On Linux, run this command to get into a Postgresql client:

```
$ sudo -u postgres psql
```

On Mac OS X, start the Postgresql server and a psql session by clicking on the elephant icon.

Within the psql session, run these commands:

```sql
CREATE DATABASE simplified_circulation_test;
CREATE DATABASE simplified_circulation_dev;

CREATE USER simplified with password '[password]';
grant all privileges on database simplified_circulation_dev to simplified;

CREATE USER simplified_test with password '[password]';
grant all privileges on database simplified_circulation_test to simplified_test;

--Add pgcrypto to any circulation manager databases.
\c simplified_circulation_dev
create extension pgcrypto;
\c simplified_circulation_test
create extension pgcrypto;
```

# Create the Python virtual environment

The next step is to create a virtual environment. To create the virtual environment, use `virtualenv`:

```
$ cd circulation # Or metadata
virtualenv -p /usr/bin/python2.7 env
```

Then add the following lines to the bottom of `env/bin/activate`:

```
export SIMPLIFIED_PRODUCTION_DATABASE="postgres://simplified:[password]@localhost:5432/simplified_circulation_dev"
export SIMPLIFIED_TEST_DATABASE="postgres://simplified_test:[password]@localhost:5432/simplified_circulation_test"
```

These environment variables will point the application server to the correct database. All further configuration will be done through the administrative interface and stored in the database.

Now, activate the virtual environment:

```
source env/bin/activate
```

# Install Python requirements into the virtual environment

This can take a very long time--over 30 minutes for the metadata wrangler.

```
pip install -r requirements.txt
```

[This page](http://www.zezuladp.com/2014/10/scaling-numpy-and-scipy-with-django-and.html) explains the problems with installing scipy (used by the metadata wrangler) through pip. I'm using pip because we are running Python 2.7 and AMI instances have Python 2.6 as the system Python.

On the metadata wrangler and the circulation manager, download the TextBlob corpora. This shouldn't be necessary on the circulation manager, but for the moment it is.

```
$ python -m textblob.download_corpora
```

# Deploy

There are lots of technologies you can use to get your server up and running. Before you do any of them, make sure you can run `python app.py` and visit `http://localhost:6500/` to see the application server running.

Done? Great! Now you can use whatever tools your dev team likes best to get the app to the people. If you don't have a preference, you might like to use our [Deployment Instructions for Nginx & uWSGI](https://github.com/NYPL-Simplified/Simplified/wiki/Deployment:-Nginx-&-uWSGI). If you run into any problems, please share them along with your solutions, so we can update the instructions.

If you use a different stack, we'd love for you to send us a setup walk-through with us so we can add it to the wiki.