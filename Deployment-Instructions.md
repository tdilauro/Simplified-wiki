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
    brew install pkg-config libffi
    # install virtualenv
    pip install virtualenv
    # Add postgres executables to PATH
    export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/9.6/bin/
    ```

Make sure to add the `export PATH` line to your `.bashrc` file -- this will let you run `psql` from the command line in the future.

6. Install additional dependencies
```
    brew install libjpeg
```

## Set up the Elasticsearch service (Circulation manager only)

The circulation manager requires on Elasticsearch version 6 or later. Other components of the system don't need Elasticsearch at all.

### Red Hat 

Follow the instructions at [Install Elasticsearch with RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html). You'll need to the official Elasticsearch repository and import its GPG key. Then you can run this command:

```
sudo yum install elasticsearch
```

### Debian

```
sudo apt-get install elasticsearch
```

### Mac OS X

```
brew cask install homebrew/cask-versions/adoptopenjdk8
brew install elasticsearch
elasticsearch-plugin install analysis-icu
```

### Afterwards 

Make sure Elasticsearch starts on bootup! Here's how to start it manually:

```
sudo service elasticsearch start
```

Check http://localhost:9200 to make sure the server is running.

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
cd circulation/api/admin
npm install
```

If you plan to do development on the administrative interface, you'll need to clone its repository rather than installing the latest release. Here's how to do that:

```
git clone git@github.com:NYPL-Simplified/circulation-web.git circulation-web
cd circulation-web
npm link
cd ../circulation/api/admin
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

```
pip install -r requirements.txt
```

On the metadata wrangler and the circulation manager, download the TextBlob corpora. This shouldn't be necessary on the circulation manager, but for the moment it is.

```
$ python -m textblob.download_corpora
```

# Deploy

There are lots of technologies you can use to get your server up and running. Before you do any of them, make sure you can run `python app.py` and visit `http://localhost:6500/` to see the application server running.

Done? Great! Now you can use whatever tools your dev team likes best to get the app to the people. If you don't have a preference, you might like to use our [Deployment Instructions for Nginx & uWSGI](https://github.com/NYPL-Simplified/Simplified/wiki/Deployment:-Nginx-&-uWSGI). If you run into any problems, please share them along with your solutions, so we can update the instructions.

If you use a different stack, we'd love for you to send us a setup walk-through with us so we can add it to the wiki.

# Initial configuration

Once you have the app server running, you can start configuring libraries and importing content, like any other use of the circulation manager ["Configuring a Demonstration Circulation Manager"](https://confluence.nypl.org/display/SIM/Quickstart%3A+Configuring+a+Demonstration+Circulation+Manager) will walk you through this process.