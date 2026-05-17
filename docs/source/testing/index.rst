Testing
=======

GCMS uses Jest for automated testing, with both unit and integration
tests organised by user requirement. This section documents the
testing strategy, how to run the test suite, and the conventions
to follow when adding new tests.

.. toctree::
   :maxdepth: 1

   running-tests
   writing-tests

Overview
--------

Tests live in the ``new_tests/`` folder at the project root, organised
into pairs of files per user requirement:

- ``urN-unit.test.js`` — unit tests for controller logic, with all
  database and external dependencies mocked
- ``urN-integration.test.js`` — integration tests that hit real API
  endpoints against a separate test database

For example, UR-2 (project creation) is covered by:

- ``ur2-unit.test.js`` — tests the controller in isolation
- ``ur2-integration.test.js`` — tests the full ``POST /api/projects``
  flow against the test database

User requirement coverage
~~~~~~~~~~~~~~~~~~~~~~~~~

Every user requirement is covered by tests. Most have both a unit and
integration test file. The exceptions are:

- **UR-7** and **UR-13** — these requirements are satisfied by code
  that does not flow through a controller, so unit tests are
  sufficient and no integration tests exist

Why both unit and integration?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unit tests verify that individual controller functions behave
correctly given a known input and mocked dependencies. They are fast
and pinpoint logic bugs precisely.

Integration tests verify that the full request-response cycle works
end-to-end — routing, middleware, controllers, and database access
all wired up together. They catch issues that unit tests miss, such
as middleware ordering, route parameter handling, and SQL errors.

Used together, they provide both depth (unit) and breadth
(integration) of coverage without duplicating effort.

Test isolation
--------------

Integration tests run against a separate test database, configured
via ``.env.test`` (see :doc:`/getting-started/environment-configuration`).
The database is recreated from scratch before each test run by
``node_scripts/install_test_db.js``, ensuring a clean state every time.

This means:

- Tests can safely create, modify, and delete records without
  affecting development or production data
- Test runs are repeatable — the same input always produces the same
  output, with no state leaking between runs
- Developers can run the full suite locally without coordinating
  with the rest of the team

Authentication bypass
~~~~~~~~~~~~~~~~~~~~~

Integration tests do not go through the Microsoft OAuth flow. Instead,
when ``NODE_ENV=test`` is set, ``app.js`` installs a middleware that
populates ``req.user`` with a stub user. This lets tests exercise
authenticated routes directly via ``supertest`` without needing a
real Microsoft session.