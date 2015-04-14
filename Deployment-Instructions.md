Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

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

# TODO: pip install uwsgi?
```

The content server also needs to have Xvfb installed:

```
# Content server only
sudo yum install xorg-x11-server-Xvfb
```

The circulation manager needs to have Elasticsearch installed:

```
# Circulation manager only
sudo rpm --import https://packages.elasticsearch.org/GPG-KEY-elasticsearch
# Install elasticsearch.repo. Instructions are here: http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html#_yum
sudo yum install elasticsearch
sudo chkconfig --add elasticsearch
```

(You can install Elasticsearch on its own server, of course; just be sure to set SEARCH_SERVER_URL appropriately.)


The metadata wrangler has a number of additional dependencies so that scikit-learn can be installed:

```
# Metadata wrangler only
sudo yum install numpy python-matplotlib ipython python-pandas sympy python-nose gcc gcc-c++ lapack-devel blas-static lapack-static
```

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

Make sure that the directory in $DATA_DIRECTORY is writable by the user that will be running scripts.

```
$ sudo chown ec2-user.ec2-user /storage
```

On the metadata wrangler, link or copy the appeal dataset into the data directory, and download the TextBlob corpora:

```
$ sudo ln -s /home/ec2-user/metadata/appeal-data /storage/appeal
$ python -m textblob.download_corpora
```

# Outstanding questions/issues

* How do we turn the instructions above into a puppet module?
* Where is a safe place to store the all-important simplified_env file, which contains credentials and secrets?
* What Jenkins jobs do we need to define? How are they to be set up?