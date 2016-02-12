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

# TODO: pip install uwsgi?
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

sudo apt-get install python-numpy
```

### On Mac:

Install homebrew: ```ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"```

Install the XCode command line tools when prompted.

Download and install Postgres.app from postgresapp.com.

Set up SSH keys using github's instructions: https://help.github.com/articles/generating-ssh-keys/


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

## Content server only
The content server also needs to have Xvfb installed:

```
# Content server only
sudo yum install xorg-x11-server-Xvfb
```

Ubuntu:
```
sudo apt-get install xvfb
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

# Check out the Simplified repositories

Depending on whether you're installing the circulation manager, the metadata wrangler, or the content server, you'll check out a different Git repository.

EITHER

```
git clone https://github.com/NYPL/Simplified-circulation.git circulation
cd circulation
```

OR

```
git clone https://github.com/NYPL/Simplified-content-server.git content
cd content
```

OR

```
git clone https://github.com/NYPL/Simplified-server.git metadata
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

```
$ sudo -u postgres psql
CREATE DATABASE simplified_circulation_test;
CREATE DATABASE simplified_circulation_dev;

CREATE USER simplified with password '[password]';
grant all privileges on database simplified_circulation_dev to simplified;

CREATE USER simplified_test with password '[password]';
grant all privileges on database simplified_circulation_test to simplified_test;
```

# Set up the data directory

On the metadata wrangler, check out the Simplified-data repository to your data directory (the directory you specified as "data_directory" in the configuration file):

```
$ git clone https://github.com/NYPL/Simplified-data.git $DATA_DIRECTORY 
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