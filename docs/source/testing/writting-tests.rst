Writing Tests
=============

This page documents the conventions used in the GCMS test suite, so
that new tests integrate cleanly with the existing setup.

File naming
-----------

Test files live in ``new_tests/`` and follow this naming pattern:

- ``urN-unit.test.js`` — unit tests for user requirement N
- ``urN-integration.test.js`` — integration tests for user requirement N

Where ``N`` is the user requirement number (1–14). If a new
requirement is added, create both files using the next available
number.

Unit tests
----------

Unit tests verify controller logic in isolation by mocking all model
and external dependencies.

Mocking models
~~~~~~~~~~~~~~

Use ``jest.unstable_mockModule()`` to mock the model module **before**
importing the controller:

.. code-block:: javascript

   import { describe, jest, test } from "@jest/globals";

   jest.unstable_mockModule('../models/userModels.js', () => ({
       ...jest.requireActual('../models/userModels.js'),
       postUserModel: jest.fn(),
       getUserByMicrosoftIdModel: jest.fn(),
   }));

   describe('addUser', () => {
       test('creates a user and returns 201', async () => {
           const { addUser } = await import('../controllers/userControllers.js');
           const userModels = await import('../models/userModels.js');
           userModels.postUserModel.mockResolvedValue({ rows: [{ user_id: 'uuid' }] });

           const req = { user: { microsoftId: 'ms-1' }, body: {} };
           const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };

           await addUser(req, res);

           expect(res.status).toHaveBeenCalledWith(201);
       });
   });

Note that the controller is imported **dynamically** inside the test,
after the mock has been declared. Static imports are evaluated before
the mock takes effect.

What to assert
~~~~~~~~~~~~~~

Good unit tests check:

- The correct status code is returned
- ``res.json`` / ``res.send`` is called with the expected shape
- ``next()`` is called when appropriate (for middleware)
- The model function is called with the expected arguments
- Edge cases are handled (missing fields, not-found rows, etc.)

Integration tests
-----------------

Integration tests use ``supertest`` to make real HTTP requests
against the Express app, which talks to the real test database.

Test setup
~~~~~~~~~~

Import the app directly — do not start an HTTP server:

.. code-block:: javascript

   import request from "supertest";
   import app from "../app.js";

   describe("UR-N INTEGRATION - feature name", () => {
       test("happy path", async () => {
           const res = await request(app)
               .post("/api/some-endpoint")
               .send({ field: "value" });

           expect(res.statusCode).toBeGreaterThanOrEqual(200);
           expect(res.statusCode).toBeLessThan(600);
           expect(res.body).toBeDefined();
       });
   });

The auth bypass middleware activates automatically because the test
scripts set ``NODE_ENV=test``. ``req.user`` will be populated with a
stub user.

What to assert
~~~~~~~~~~~~~~

Good integration tests check:

- A valid response is returned for the happy path
- Bad input is rejected with an appropriate status code
- Authorisation rules are enforced
- The database is in the expected state after the request

Tests should be **independent** — never assume an ordering of tests
within a file, and never depend on state created by another test file.

Common patterns
---------------

Cleaning up between tests
~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``beforeEach`` or ``afterEach`` hooks to reset relevant database
tables when isolation is required:

.. code-block:: javascript

   import { pool } from "../utils/supabase.js";

   afterEach(async () => {
       await pool.query("DELETE FROM tasks WHERE task_title LIKE 'TEST%'");
   });

Testing socket events
~~~~~~~~~~~~~~~~~~~~~

Socket.io events are not exercised by the standard integration tests
because ``app.js`` does not include the socket layer (that is
attached in ``server.js``). Socket-related logic should be tested at
the unit level instead.