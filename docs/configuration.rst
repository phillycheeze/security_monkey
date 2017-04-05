=============
Configuration
=============

IAM Permissions
===============

- For AWS, please see `AWS IAM instructions <./iam_aws.rst>`.
- For GCP, please see `GCP IAM instructions <./iam_gcp.rst>`.


Database
========

Security Monkey needs a postgres database.  Select one of the following:

- `Local Postgres`
- `Postgres on AWS RDS <./psq_aws.rst>`.
- `Postgres on GCP's Cloud SQL <./psq_gcp.rst>`.

Security Monkey Configuration
=============================

Most of Security Monkey's configuration is done via the Security Monkey Configuration file see: :doc:`configuration options <./options>` for a full list of options.

The default config includes a few values that you will need to change before starting Security Monkey the first time. see: security_monkey/env-config/config-deploy.py




