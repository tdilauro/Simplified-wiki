Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

# Install prerequisite packages

```
# Install packaged requirements.
sudo yum install python27 python27-devel git postgresql python-nose python-sqlalchemy python-pip nginx
sudo pip install virtualenv virtualenvwrapper

# These are necessary for installing python lxml through pip.
sudo yum install libxml2-devel libxslt-devel gcc libxslt-python

# This is necessary for installing psycopg2 through pip
sudo yum install postgresql-devel

# Used by pillow to generate JPG thumbnails.
sudo yum install libjpeg libjpeg-devel
```

# Content server-specific packages

The content server also needs to have Xvfb and Java installed:

```
# Content server only
sudo yum install java
sudo yum install xorg-x11-server-Xvfb
```

# Circulation manager-specific packages

The circulation manager needs to have Elasticsearch installed:

```
# Circulation manager only
sudo rpm --import https://packages.elasticsearch.org/GPG-KEY-elasticsearch
# Install elasticsearch.repo. Instructions are here: http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html#_yum
sudo yum install elasticsearch
sudo chkconfig --add elasticsearch

# Install the AWS discovery plugin
sudo /usr/share/elasticsearch/bin/plugin install elasticsearch/elasticsearch-cloud-aws/2.5.0
```

See the /etc/elasticsearch/elasticsearch.yml config file on the circulation manager machines for an example of using unicast to create a cluster between several different circulation machines. Here, the machine at 10.225.128.160 is set up to make a cluster with the machine at 10.225.130.107:

```
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["10.225.130.107"]
```

Each circulation manager must have its own Elasticsearch server, and the servers must form a cluster. Each circulation manager must set the SEARCH_SERVER_URL environment variable to point to its own external-facing IP address. For example, the machine at 10.225.128.160 has SEARCH_SERVER_URL set to "http://10.225.128.160:9200/"

# Metadata wrangler-specific packages

The metadata wrangler has a number of additional dependencies so that scikit-learn can be installed:

```
# Metadata wrangler only
sudo yum install numpy python-matplotlib ipython python-pandas sympy python-nose gcc gcc-c++ lapack-devel blas-static lapack-static
```

# Check out the repo

Check out the appropriate repository:

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

# Create a virtual environment

```
# Create a virtual environment
virtualenv -p /usr/bin/python2.7 env
# Manual step: install env/bin/simplified_env and add the following line to the bottom of env/bin/activate:
# source simplified_env
source env/bin/activate

# Install Python requirements into the virtual environment.
# This can take a very long time--over 30 minutes for the metadata wrangler.
pip install -r requirements.txt
```

[This page](http://www.zezuladp.com/2014/10/scaling-numpy-and-scipy-with-django-and.html) explains the problems with installing scipy (used by the metadata wrangler) through pip. I'm using pip because we are running Python 2.7 and AMI instances have Python 2.6 as the system Python.

# Data setup

```
# Download the TextBlob corpora
source env/bin/activate
python -m textblob.download_corpora
```

On the metadata wrangler, check out Simplified-data as $DATA_DIRECTORY:

```
git clone https://github.com/NYPL/Simplified-data.git $DATA_DIRECTORY
```

Make sure that $DATA_DIRECTORY is writable by the user that will be running scripts.

```
$ sudo chown ec2-user.ec2-user $DATA_DIRECTORY
```

# Running

```

# NOTE: This assumes you have already run 'source env/bin/activate' from
# your current shell!
$ env/bin/uwsgi --ini uwsgi.ini --touch-reload=uwsgi.ini --reload-on-exception --buffer-size=131072 &
```