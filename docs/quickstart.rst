=================
Quick Start Guide
=================

*******************
Setup on AWS or GCP
*******************

Security Monkey can run on an Amazon EC2 (AWS) instance or a Google Cloud Platform (GCP) instance (Google Cloud Platform). The only real difference in the installation is the IAM configuration and the bringup of the Virtual Machine that runs Security Monkey.

***************
IAM Permissions
***************

- For AWS, please see `AWS IAM instructions <./iam_aws.rst>`.
- For GCP, please see `GCP IAM instructions`.

.. _GCP IAM instructions: iam_gcp.rst

********
Database
********

Security Monkey needs a postgres database.  Select one of the following:

- `Local Postgres`
- `Postgres on AWS RDS <./psq_aws.rst>`.
- `Postgres on GCP's Cloud SQL <./psq_gcp.rst>`.

Local Postgres
==============

Install Posgres::

    sudo apt-get install ...

Configure the DB::

    sudo -u postgres psql
    CREATE DATABASE "secmonkey";
    CREATE ROLE "securitymonkeyuser" LOGIN PASSWORD 'securitymonkeypassword';
    CREATE SCHEMA secmonkey;
    GRANT Usage, Create ON SCHEMA "secmonkey" TO "securitymonkeyuser";
    set timezone TO 'GMT';
    select now();
    \q


*******************
Launch an Instance:
*******************

- `docker instructions <./docker.html>`.
- `Launch an AWS instance <./instance_launch_aws.rst>`.
- `Launch a GCP instance <./instance_launch_gcp.rst>`.


****************************************
Install Security Monkey on your Instance
****************************************

Installation Steps:

- `Prerequisites`
- `Clone security_monkey`
- `Compile (or Download) the web UI`
- Review the config`

Prerequisites
=============

We now have a fresh install of Ubuntu.  Let's add the hostname to the hosts file::

    $ hostname
    ip-172-30-0-151

Add this to /etc/hosts: (Use nano if you're not familiar with vi.)::

    $ sudo vi /etc/hosts
    127.0.0.1 ip-172-30-0-151

Create the logging folders::

    sudo mkdir /var/log/security_monkey
    sudo mkdir /var/www
    sudo chown www-data /var/log/security_monkey
    sudo chown www-data /var/www

Let's install the tools we need for Security Monkey::

    $ sudo apt-get update
    $ sudo apt-get -y install python-pip python-dev python-psycopg2 postgresql postgresql-contrib libpq-dev nginx supervisor git libffi-dev gcc python-virtualenv

Clone security_monkey
=====================

Releases are on the master branch and are updated about every three months.  Bleeding edge features are on the develop branch.

    cd /usr/local/src
    sudo git clone --depth 1 --branch master https://github.com/Netflix/security_monkey.git
    cd security_monkey
    sudo virtualenv venv
    sudo pip install --upgrade setuptools
    sudo python setup.py install

Fix ownership for python modules::

    sudo usermod -a -G staff www-data
    sudo chgrp staff /usr/local/lib/python2.7/dist-packages/*.egg

Compile (or Download) the web UI
================================

If you're using the stable (master) branch, you have the option of downloading the web UI instead of compiling it.  Visit `the latest release <https://github.com/Netflix/security_monkey/releases/latest>` and download static.tar.gz.

If you're using the bleeding edge (develop) branch, you will need to compile the web UI by following these instructions::

    # Get the Google Linux package signing key.
    $ curl https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

    # Set up the location of the stable repository.
    cd ~
    curl https://storage.googleapis.com/download.dartlang.org/linux/debian/dart_stable.list > dart_stable.list
    sudo mv dart_stable.list /etc/apt/sources.list.d/dart_stable.list
    sudo apt-get update
    sudo apt-get install -y dart

    # Build the Web UI
    cd /usr/local/src/security_monkey/dart
    sudo /usr/lib/dart/bin/pub get
    sudo /usr/lib/dart/bin/pub build

    # Copy the compiled Web UI to the appropriate destination
    sudo mkdir -p /usr/local/src/security_monkey/security_monkey/static/
    sudo /bin/cp -R /usr/local/src/security_monkey/dart/build/web/* /usr/local/src/security_monkey/security_monkey/static/
    sudo chgrp -R www-data /usr/local/src/security_monkey

Configure the Application
=========================

Security Monkey ships with a config for this quickstart guide called config-quickstart.py.  This config assumes that you are using the local db option.  If you setup AWS RDS or GCP Cloud SQL as your database, you will need to modify the SQLACHEMY_DATABASE_URI to point to your DB.

For an explanation of the configuration options, `see options <./options.rst>`.


SECURITY_MONKEY_SETTINGS
------------------------

The SECURITY_MONKEY_SETTINGS environment variable points to config-quickstart.py.::

    $ export SECURITY_MONKEY_SETTINGS=/usr/local/src/security_monkey/env-config/config-quickstart.py

Create the database tables:
===========================

Security Monkey uses Flask-Migrate (Alembic) to keep database tables up to date.  To create the tables, run  this command::

    cd /usr/local/src/security_monkey/
    sudo -E python manage.py db upgrade

*********************************************
Populate Security Monkey with Accounts
*********************************************

Add Amazon Accounts
===================

This will add Amazon owned AWS accounts to security monkey. ::

    $ sudo -E python manage.py amazon_accounts

Add Your AWS/GCP Accounts
=========================

You'll need to add at least one account before starting the scheduler.  It's easiest to add them from the command line, but it can also be done through the web UI. ::

    $ sudo -E python manage.py add_account_aws 
    $ sudo -E python manage.py add_account_gcp

Create the first user:
======================

Users can be created on the command line or by registering in the web UI::

    $ sudo -E python manage.py create_user "you@youremail.com" "Admin"
    > Password:
    > Confirm Password:

create_user takes two parameters.  1) is the email address and 2) is the role.

Roles should be one of these:

- View
- Comment
- Justify
- Admin

*********************
Setting up Supervisor
*********************

Supervisor will auto-start security monkey and will auto-restart security monkey if
it crashes.

Copy supervisor config::

    sudo cp /usr/local/src/security_monkey/supervisor/security_monkey.conf /etc/supervisor/conf.d/security_monkey.conf
    sudo service supervisor restart
    sudo supervisorctl status

Supervisor will attempt to start two python jobs and make sure they are running.  The first job, securitymonkey,
is gunicorn, which it launches by calling manage.py run_api_server.

The second job supervisor runs is the scheduler, which polls for changes.

You can track progress by tailing /var/log/security_monkey/securitymonkey.log.

*************************
Create an SSL Certificate
*************************

For this quickstart guide, we will use a self-signed SSL certificate.  In production, you will want to use a certificate that has been signed by a trusted certificate authority.::

    $ cd ~

There are some great instructions for generating a certificate on the Ubuntu website:

`Ubuntu - Create a Self Signed SSL Certificate <https://help.ubuntu.com/12.04/serverguide/certificates-and-security.html>`_

The last commands you need to run from that tutorial are in the "Installing the Certificate" section:

.. code-block:: bash

    sudo cp server.crt /etc/ssl/certs
    sudo cp server.key /etc/ssl/private

Once you have finished the instructions at the link above, and these two files are in your /etc/ssl/certs and /etc/ssl/private, you are ready to move on in this guide.

************
Setup Nginx:
************

Security Monkey uses gunicorn to serve up content on its internal 127.0.0.1 address.  For better performance, and to offload the work of serving static files, we wrap gunicorn with nginx.  Nginx listens on 0.0.0.0 and proxies some connections to gunicorn for processing and serves up static files quickly.

securitymonkey.conf
===================

Copy the config file into place::

        sudo cp /usr/local/src/security_monkey/nginx/security_monkey.conf /etc/nginx/sites-available/security_monkey.conf

Symlink the sites-available file to the sites-enabled folder::

    $ sudo ln -s /etc/nginx/sites-available/security_monkey.conf /etc/nginx/sites-enabled/security_monkey.conf

Delete the default configuration::

    $ sudo rm /etc/nginx/sites-enabled/default

Restart nginx::

    $ sudo service nginx restart

*******************
Logging into the UI
*******************

You should now be able to reach your server

.. image:: images/resized_login_page-1.png

**********
User Guide
**********

See the `User Guide <./userguide.rst>` for a walkthrough of the security_monkey features.

**********
Contribute
**********

It's easy to extend security_monkey with new rules or new technologies.  If you have a good idea, **please send us a pull request**.  I'll be delighted to include them.
