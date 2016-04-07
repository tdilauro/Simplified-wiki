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
sudo apt-get install libffi-dev

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
    # install python 2.7
    brew install python
    # install easy_install
    curl https://bootstrap.pypa.io/ez_setup.py -o - | python
    # install pip
    easy_install pip
    # install virtualenv
    pip install virtualenv
    # Add postgres executables to PATH
    export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/9.4/bin/
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

## Circulation manager only
Set up the Elasticsearch service

```
sudo apt-get install openjdk-7-jre
```
Follow the instructions here: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html

Or, on a brew-capable Mac:
```
  $ brew tap caskroom/cask
  $ brew install brew-cask
  $ brew cask install java
  $ brew install homebrew/versions/elasticsearch17
```

# Check out the Simplified repositories

Depending on whether you're installing the circulation manager, the metadata wrangler, or the content server, you'll check out a different Git repository.

EITHER

```
git clone https://github.com/NYPL-Simplified/circulation.git circulation
cd circulation
```

OR

```
git clone https://github.com/NYPL-Simplified/content_server.git content
cd content
```

OR

```
git clone https://github.com/NYPL-Simplified/metadata_wrangler.git metadata
cd metadata
```

Then:

```
# Initialize submodules
git submodule init
git submodule update
```

# Create the Python virtual environment

The next step is to create a virtual environment and set up your [[configuration file|Configuration]]. To create the virtual environment, use `virtualenv`:

```
$ cd circulation # Or metadata, or content
virtualenv -p /usr/bin/python2.7 env
```

Then create your [[configuration file|Configuration]].

Finally, add the following line to the bottom of `env/bin/activate`:

```
export SIMPLIFIED_CONFIGURATION_FILE="/full/path/to/configuration/file.json"
```

where `/full/path/to/configuration/file.json` is the full path to your configuration file.

For the content manager, also add these two lines to `activate`:

```
Xvfb :1 &
export DISPLAY=:1
```

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

# Set up database users

On Linux, run this command to get into a Postgresql client:

```
$ sudo -u postgres psql
```

On Mac OS X, start the Postgresql server and a psql session by clicking on the elephant icon.

Within the psql session, run these commands:

```
CREATE DATABASE simplified_circulation_test;
CREATE DATABASE simplified_circulation_dev;

CREATE USER simplified with password '[password]';
grant all privileges on database simplified_circulation_dev to simplified;

CREATE USER simplified_test with password '[password]';
grant all privileges on database simplified_circulation_test to simplified_test;
```

# Set up the data directory (metadata and content only)

On the metadata wrangler, check out the Simplified-data repository to your data directory (the directory you specified as "data_directory" in the configuration file):

```
$ git clone https://github.com/NYPL-Simplified/data.git $DATA_DIRECTORY 
```

On the content server, you can just create an empty directory at $DATA_DIRECTORY

On the metadata wrangler and the content server, make sure that $DATA_DIRECTORY is writable by the user that will be running scripts.

```
$ sudo chown ec2-user.ec2-user $DATA_DIRECTORY
```

On the metadata wrangler and the circulation manager, download the TextBlob corpora. This shouldn't be necessary on the circulation manager, but for the moment it is.

```
$ python -m textblob.download_corpora
```

# Stand Up Your Server

There are lots of technologies you can use to get your server up and running. Before you do any of them, make sure you can run `python app.py` and get the app going locally.

Done? Great! Now you can use whatever tools your dev team likes best to get the app to the people. If you don't have a preference, you might like to use our [Deployment Instructions for Nginx & uWSGI](https://github.com/NYPL-Simplified/Simplified/wiki/Deployment:-Nginx-&-uWSGI). If you run into any problems, please share them along with your solutions, so we can update the instructions.

If you use a different stack, we'd love for you to send us a setup walk-through with us so we can add it to the wiki.