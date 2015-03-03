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

The metadata wrangler has a number of additional dependencies so that scikit-learn can be installed:

```
# Metadata wrangler only
sudo yum install numpy python-matplotlib ipython python-pandas sympy python-nose gcc gcc-c++ lapack-devel
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

# Install Python requirements into the virtual environment. This can take a very long time.
pip install -r requirements.txt

```

Make sure that the directory in $DATA_DIRECTORY is writable by the user that will be running scripts.

```
$ chown ec2-user.ec2-user /storage
```

On the metadata wrangler, link or copy the appeal dataset into the data directory:

```
$ ln -s /home/ec2-user/metadata/appeal-data /storage/appeal
```

# Outstanding questions/issues

* How do we turn the instructions above into a puppet module?
* Where is a safe place to store the all-important simplified_env file, which contains credentials and secrets?
* What Jenkins jobs do we need to define? How are they to be set up?