=====================
Atlas Data Lake Tests
=====================

.. contents::

----

Introduction
============

The YAML and JSON files in this directory tree are platform-independent tests
that drivers can use to assert compatibility with `Atlas Data Lake <https://docs.mongodbcom/datalake>`_.

Running these integration tests will require a running ``mongohoused``. See the
README file at ``https://github.com/10gen/mongohouse`` for more information.


Test Format
===========

The keys found in the YAML files for the connection tests are a subset of the keys found in
the `Test Plan for Connection Strings <../../connection-string/tests/README.rst#Format>`_.
The keys found in the YAML files for the CRUD tests are a subset of the keys found in the
`Test Plan for CRUD <../../crud/tests/README.rst#Test-Format>`_.
