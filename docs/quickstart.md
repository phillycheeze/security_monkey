Quick Start Guide
=================

Setup on AWS or GCP
-------------------

Security Monkey can run on an Amazon EC2 (AWS) instance or a Google Cloud Platform (GCP) instance (Google Cloud Platform). The only real difference in the installation is the IAM configuration and the bringup of the Virtual Machine that runs Security Monkey.

IAM Permissions
---------------

-   For GCP, please see [GCP IAM instructions].

Database
--------

Security Monkey needs a postgres database. Select one of the following:

-   Local Postgres
-   Postgres on AWS RDS &lt;./psq\_aws.rst&gt;.
-   Postgres on GCP's Cloud SQL &lt;./psq\_gcp.rst&gt;.

### Local Postgres

Install Posgres:

    sudo apt-get install ...

Configure the DB:

    sudo -u postgres psql
    CREATE DATABASE "secmonkey";
    CREATE ROLE "securitymonkeyuser" LOGIN PASSWORD 'securitymonkeypassword';
    CREATE SCHEMA secmonkey;
    GRANT Usage, Create ON SCHEMA "secmonkey" TO "securitymonkeyuser";
    set timezone TO 'GMT';
    select now();
    \q

Launch an Instance:
-------------------

-   docker instructions &lt;./docker.html&gt;.
-   Launch an AWS instance &lt;./instance\_launch\_aws.rst&gt;.
-   Launch a GCP instance &lt;./instance\_launch\_gcp.rst&gt;.

Install Security Monkey on your Instance
----------------------------------------

Installation Steps:

-   Prerequisites
-   Clone security\_monkey
-   Compile (or Download) the web UI
-   Review the config\`

### Prerequisites

We now have a fresh install of Ubuntu. Let’s add the hostname to the hosts file:

    $ hostname
    ip-172-30-0-151

Add this to /etc/hosts: (Use nano if you’re not familiar with vi.):

    $ sudo vi /etc/hosts
    127.0.0.1 ip-172-30-0-151

Create the logging folders:

    sudo mkdir /var/log/security_monkey
    sudo mkdir /var/www
    sudo chown www-data /var/log/security_monkey
    sudo chown www-data /var/www

Let’s install the tools we need for Security Monkey:

    $ sudo apt-get update
    $ sudo apt-get -y install python-pip python-dev python-psycopg2 postgresql postgresql-contrib libpq-dev nginx supervisor git libffi-dev gcc python-virtualenv

### Clone security\_monkey

Releases are on the master branch and are updated about every three months. Bleeding edge features are on the develop branch.

> cd /usr/local/src sudo git clone –depth 1 –branch master <https://github.com/Netflix/security_monkey.git> cd security\_monkey sudo virtualenv venv sudo pip install –upgrade setuptools sudo python setup.py install

Fix ownership for python modules:

    sudo usermod -a -G staff www-data
    sudo chgrp staff /usr/local/lib/python2.7/dist-packages/*.egg

### Compile (or Download) the web UI

If you’re using the stable (master) branch, you have the option of downloading the web UI instead of compil

  [GCP IAM instructions]: iam_gcp.md