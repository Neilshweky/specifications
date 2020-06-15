=====================
Atlas Data Lake Tests
=====================

.. contents::

----

Introduction
============

The YAML and JSON files in this directory tree are platform-independent tests
that drivers can use to prove their conformance to the Atlas Data Lake spec.

Running these integration tests will require a running ``mongohoused``. Requirements
for building ``mongohoused`` are as follows:

  - Go version 1.13
  - Building ``mqlrun``

See the README file at ``https://github.com/10gen/mongohouse`` for more information.

Building ``mongohoused`` locally
================================

To build ``mongohoused`` for local testing, follow these steps:

  - Install Go version 1.13 or higher.
  - Set up the environment variables for Go::
      export GOROOT="<path to Go installation>"
      export PATH="${GOROOT}/bin:$PATH"
      export GOPATH=`pwd`/.gopath
  - Clone the ``mongohouse`` repository::
      git clone git@github.com:10gen/mongohouse.git
      cd mongohouse
  - Set the Go modules environment variable::
      GO111MODULE=on go mod download
  - Build ``mqlrun``::
    ./build.sh tools:download:mqlrun
    export MONGOHOUSE_MQLRUN=`pwd`/artifacts/mqlrun
  - Build ``mongohoused``::
      ./build.sh build:mongohoused
  - Run ``mongohoused`` locally::
      ./artifacts/mongohoused --config ./testdata/config/single-local-local.yaml


Test Format
===========

The keys found in the YAML files for the connection tests are a subset of the keys found in
the `Test Plan for Connection Strings <../../connection-string/tests/README.rst#Format>`_.
The keys found in all other YAML files are a subset of the keys found in the
`Test Plan for CRUD <../../crud/tests/README.rst#Test-Format>`_.


Test Runner Implementation
==========================

This section provides guidance for implementing a test runner.

Before running the tests:

- Create a global MongoClient (``globalMongoClient``) and connect to ``mongohoused``.

For each test file:

- Determine the collection and database under test, utilizing the top-level
  ``collection_name`` and/or ``database_name`` fields if present.

- For each element in the ``tests`` array:

  - Create a local MongoClient (``localMongoClient``) and connect to the
    ``mongohoused``. This client will be used for executing the test case.

  - Activate command monitoring for ``localMongoClient`` and begin capturing
    events.

  - For each element in the ``operations`` array:

    - Using ``localMongoClient``, select the appropriate ``object`` to execute
      the operation. Default to the collection under test if this field is not
      present.

    - Given the ``name`` and ``arguments``, execute the operation on the object
      under test. Capture the result of the operation, if any, and observe
      whether an error occurred. If an error is encountered that includes a
      result, extract the result object.

    - If ``error`` is present and true, assert that the operation encountered an
      error. Otherwise, assert that no error was encountered.

    - if ``result`` is present, assert that it matches the operation's result.

  - Deactivate command monitoring for ``localMongoClient``.

  - If the ``expectations`` array is present, assert that the sequence of
    emitted CommandStartedEvents from executing the operation(s) matches the
    sequence of ``command_started_event`` objects in the ``expectations`` array.

Evaluating Matches
------------------

The expected values for results (e.g. ``result`` for an operation
operation, ``command_started_event.command``, elements in ``outcome.data``) are
written in `Extended JSON <../../extended-json.rst>`_. Drivers may adopt any of
the following approaches to comparisons, as long as they are consistent:

- Convert ``actual`` to Extended JSON and compare to ``expected``
- Convert ``expected`` and ``actual`` to BSON, and compare them
- Convert ``expected`` and ``actual`` to native representations, and compare
  them

Extra Fields in Actual Documents
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When comparing ``actual`` and ``expected`` *documents*, drivers should permit
``actual`` documents to contain additional fields not present in ``expected``.
For example, the following documents match:

- ``expected`` is ``{ "x": 1 }``
- ``actual`` is ``{ "_id": { "$oid" : "000000000000000000000001" }, "x": 1 }``

In this sense, ``expected`` may be a subset of ``actual``. It may also be
helpful to think of ``expected`` as a form of query criteria. The intention
behind this rule is that it is not always feasible for the test to express all
fields in the expected document(s) (e.g. session and cluster time information
in a ``command_started_event.command`` document).

This rule for allowing extra fields in ``actual`` only applies for values that
correspond to a document. For instance, an actual result of ``[1, 2, 3, 4]`` for
a ``distinct`` operation would not match an expected result of ``[1, 2, 3]``.
Likewise with the ``find`` operation, this rule would only apply when matching
documents *within* the expected result array and actual cursor.
