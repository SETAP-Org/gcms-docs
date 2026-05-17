Running Tests
=============

GCMS exposes a single NPM script for running the full test suite,
plus two helper scripts for triggering email and notification flows
manually.

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

Runs every test file in ``new_tests/`` with coverage reporting,
after resetting the test database from scratch.

::

   npm test

The script does three things in order:

1. ``cross-env NODE_ENV=test node node_scripts/install_test_db.js``
   — drops and recreates the test database, then applies
   ``db/schema.sql`` and ``db/seed_dev.sql``
2. ``cross-env NODE_ENV=test`` — sets the environment so that
   ``app.js`` loads ``.env.test`` and installs the auth bypass
   middleware
3. ``jest --runInBand --coverage`` — runs the test files
   sequentially (avoiding database contention) and generates a
   coverage report, excluding model files, scripts, and static
   client-side code from coverage measurement

Manual flow scripts
-------------------

In addition to the automated test suite, two NPM scripts trigger
specific real-world flows for manual verification. These do not run
under Jest and do not produce a coverage report.

``npm run email:test``
~~~~~~~~~~~~~~~~~~~~~~

Runs ``testEmail.mjs``, which sends a sample notification email via
Mailtrap using the credentials in ``.env.mailconfig``. Use this to
verify that SMTP credentials are valid without going through the
full notification trigger path.

``npm run notification:test``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Runs ``testNotification.mjs``, which exercises the end-to-end
notification flow including database insertion, Socket.io emission,
and email delivery. Useful when validating that all parts of the
notification pipeline are wired correctly.

Coverage report
---------------

After ``npm test`` runs, Jest writes a coverage report to
``coverage/`` in the project root. Open
``coverage/lcov-report/index.html`` in a browser for an interactive
view of which lines and branches were exercised by the test suite.

Coverage is reported per file and per metric (statements, branches,
functions, lines). Aim for at least 80% line coverage on any new
controller you add.

The following are intentionally excluded from coverage measurement:

- ``models/`` — covered by integration tests rather than unit tests
- ``node_scripts/`` — standalone setup utilities
- ``listen.js`` — single-line server boot
- ``public/scripts/`` — client-side JS, not in scope for backend testing

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

Jest may be waiting on an open database connection or socket. If you
have added a new integration test, make sure any open resources are
closed in an ``afterAll`` hook.

If unit tests can't find a mock
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unit tests use ``jest.unstable_mockModule()`` for ESM imports, which
must be declared **before** the dynamic ``import()`` of the module
under test. See an existing unit test file for the correct pattern.