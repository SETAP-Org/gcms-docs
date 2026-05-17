Running Tests
=============

GCMS exposes two NPM scripts for running tests. Both set
``NODE_ENV=test`` automatically and reset the test database before
running.

Prerequisites
-------------

Before running tests, make sure:

- All env files are in place, including ``.env.test``
  (see :doc:`/getting-started/environment-configuration`)
- The test database referenced by ``.env.test`` exists and is
  reachable
- ``npm install`` has been run to install Jest and its dependencies

Available scripts
-----------------

``npm test``
~~~~~~~~~~~~

Runs a focused subset of tests with coverage reporting. Currently
configured to run the UR-11 unit and integration tests as a smoke
test before the full suite.

::

   npm test

Use this when you want quick feedback during development.

``npm run test-all``
~~~~~~~~~~~~~~~~~~~~

Runs every test file in ``new_tests/`` with coverage reporting.

::

   npm run test-all

Use this before pushing changes or opening a pull request, to confirm
nothing has regressed across the full suite.

How the scripts work
--------------------

Both scripts perform the same three-step process:

1. ``cross-env NODE_ENV=test`` — sets the environment so that
   ``app.js`` loads ``.env.test`` and installs the auth bypass
   middleware
2. ``node node_scripts/install_test_db.js`` — drops and recreates the
   test database, then applies ``db/schema.sql`` and ``db/seed_dev.sql``
3. ``jest --runInBand --forceExit --coverage`` — runs the test files
   sequentially (avoiding database contention) and generates a
   coverage report

Coverage report
---------------

After a test run, Jest writes a coverage report to ``coverage/`` in
the project root. Open ``coverage/lcov-report/index.html`` in a
browser for an interactive view of which lines and branches were
exercised by the test suite.

Coverage is reported per file and per metric (statements, branches,
functions, lines). Aim for at least 80% line coverage on any new
controller you add.

Troubleshooting
---------------

If tests fail with database errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The test database may not have been recreated cleanly. Check that
``DB_URL`` in ``.env.test`` points to a database the current user
can drop and recreate (Supabase service role accounts can; local
PostgreSQL users may need elevated privileges).

If integration tests hang
~~~~~~~~~~~~~~~~~~~~~~~~~

Jest may be waiting on an open database connection. The ``--forceExit``
flag in the scripts should prevent this, but if you have added a new
integration test, make sure any open connections or sockets are
closed in an ``afterAll`` hook.

If unit tests can't find a mock
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unit tests use ``jest.unstable_mockModule()`` for ESM imports, which
must be declared **before** the dynamic ``import()`` of the module
under test. See an existing unit test file for the correct pattern.