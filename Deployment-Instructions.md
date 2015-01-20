Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

```
# Install packaged requirements.
sudo yum install git postgresql python-nose python-sqlalchemy python-pip 
# These are necessary for installing python lxml through pip
sudo yum install python-devel libxml2-devel libsxlt-devel gcc
# This is necessary for installing psycopg2 through pip
sudo yum install postgresql-devel

# libxslt-python may also be necessary

sudo pip install virtualenv virtualenvwrapper

# Check out the repository and install Python requirements.
git clone https://github.com/NYPL/Simplified-circulation.git
cd Simplified-circulation
git submodule init
git submodule update
pip install -r requirements.txt

# Create a virtual environment
virtualenv env
# Manual step: install env/bin/simplified_env and add the following line to the bottom of env/bin/activate:
# source simplified_env
source env/bin/activate


```