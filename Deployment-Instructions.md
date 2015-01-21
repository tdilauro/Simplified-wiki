Step-by-step instructions for deploying any of the Simplified applications on an EC2 AMI.

```
# Install packaged requirements.
sudo yum install git postgresql python-nose python-sqlalchemy python-pip 
sudo pip install virtualenv virtualenvwrapper

# These are necessary for installing python lxml through pip.
# libxslt-python may also be necessary, I'm not sure.
sudo yum install python-devel libxml2-devel libsxlt-devel gcc

# This is necessary for installing psycopg2 through pip
sudo yum install postgresql-devel
```

Check out the appropriate repository:

```
git clone https://github.com/NYPL/Simplified-circulation.git circulation
cd circulation
```

OR

```
git clone https://github.com/NYPL/Simplified-content.git content
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
virtualenv env
# Manual step: install env/bin/simplified_env and add the following line to the bottom of env/bin/activate:
# source simplified_env
source env/bin/activate

# Install Python requirements into the virtual environment
pip install -r requirements.txt

```

# Outstanding questions/issues

* How do we turn the instructions above into a puppet module?
* Where is a safe place to store the all-important simplified_env file, which contains credentials and secrets?
* What Jenkins jobs do we need to define? How are they to be set up?
* Is it still useful to run the server in a python virtualenv? Or does it not matter since these are single-purpose servers?
* What are the AWS credentials that will let the content server and the metadata wrangler upload static content to S3?
* Instead of a single 'libsimple' database I need three databases, one for each component.
* What exactly is the dev environment 'for'? How about the QA environment? What's my development workflow?
* Can we connect the dev environment to the ILS to perform patron authentication? If not, can we connect to a sandbox ILS instead?

## Database for unit tests

I need a transient database to use when running the unit test suite in the development environment. This would be a database that I can destroy and re-create if something goes wrong during a test.

1. I set up postgresql servers on the development machines and create/destroy databases as needed.
2. Test database managed on the same terms as the dev/qa/production database. Instead of getting to a known state by dropping and recreating the database itself, I do it by dropping all the tables.
3. I change the tests to use sqlite instead of postgres.