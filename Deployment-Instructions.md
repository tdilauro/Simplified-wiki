Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

```
sudo yum install git postgresql python-nose python-sqlalchemy python-pip
sudo pip install virtualenv virtualenvwrapper
cd ~/
mkdir ~/.virtualenv
cd ~/.virtualenv
virtualenv default
# Manual step: edit activate to set environment variables
source ~/.virtualenv/default/bin/activate
```