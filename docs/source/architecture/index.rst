Architecture
============

GCMS is built as a layered Node.js application following a classic
Model-Controller pattern, served by Express with real-time updates
powered by Socket.io. This page documents the major components, how
they fit together, and how a request flows through the system.

High-level overview
-------------------

.. mermaid::

   flowchart TB
       Browser["Browser<br/>(EJS views + JS)"]
       Server["server.js<br/>(Express + Socket.io)"]
       Routes["routes/<br/>(URL → controller)"]
       Controllers["controllers/<br/>(request handling)"]
       Models["models/<br/>(SQL queries)"]
       Utils["utils/<br/>(shared helpers)"]
       DB[("Supabase<br/>PostgreSQL + Storage")]
       External["External services<br/>Microsoft Graph<br/>Google Gemini<br/>Mailtrap"]

       Browser <--> Server
       Server --> Routes
       Routes --> Controllers
       Controllers --> Models
       Controllers --> Utils
       Models --> DB
       Utils --> DB
       Utils --> External

Project layout
--------------

The repository is organised by responsibility, not by feature:

============================  ====================================================
Folder                        Purpose
============================  ====================================================
``routes/``                   Express route definitions; map URLs to controllers
``controllers/``              Request handlers; validate input, call models, send responses
``models/``                   SQL queries and database access logic
``utils/``                    Cross-cutting concerns (auth, sessions, sockets, email, AI)
``views/``                    EJS templates rendered server-side
``public/``                   Static assets (CSS, client-side JS, images)
``db/``                       Schema definition and seed scripts
``node_scripts/``             Standalone Node scripts (e.g. ``install_db.js``)
``Tests/``                    Jest unit tests, one file per user requirement
``Automated_Testing/``        Standalone automated test scripts (e.g. notification triggers)
============================  ====================================================

The layered structure means most features touch one file per layer.
For example, the task feature uses ``routes/taskRoutes.js``,
``controllers/taskControllers.js``, and ``models/taskModels.js``.

Request flow
------------

A typical HTTP request flows through the application like this:

1. **Browser** sends a request (e.g. ``POST /api/tasks``)
2. **server.js** routes it to the correct ``app.use`` mount
3. **Route file** matches the URL pattern and calls the controller
4. **Controller** validates the request, checks authentication,
   and calls one or more models
5. **Model** executes a parameterised SQL query against Supabase
6. **Controller** formats the result and sends an HTTP response
7. **Socket.io** (where relevant) broadcasts the change to other
   connected clients in the same project

server.js
---------

``server.js`` is the application entry point. It:

- Loads environment variables from ``.env.auth``
- Configures Express middleware (JSON parsing, cookies, EJS templating,
  static assets)
- Sets up sessions and authentication via the ``utils/`` helpers
- Initialises the Socket.io server
- Mounts each route file under the appropriate path (``/`` for page
  routes, ``/api`` for everything else)
- Starts the HTTP server on port 3000

All other concerns are delegated to the appropriate layer.

Routes
------

Route files (in ``routes/``) define the URL surface of the application.
Each route file groups related endpoints — for example
``taskRoutes.js`` handles all task-related URLs.

There are two categories of routes:

**Page routes** (``pageRoutes.js``)
   Server-side rendered EJS views, mounted at ``/``. Examples:
   ``/``, ``/projects``, ``/projects/:id``, ``/profile``.

**API routes** (everything else)
   JSON endpoints, mounted at ``/api``. Used by client-side JS for
   asynchronous operations. Grouped by resource:

   - ``authRoutes.js`` — Microsoft OAuth flow
   - ``userRoutes.js`` — user profile and account operations
   - ``projectRoutes.js`` — project CRUD
   - ``taskRoutes.js`` — task CRUD and assignment
   - ``calendarRoutes.js`` — meetings and Microsoft calendar sync
   - ``notificationsRoutes.js`` — notification fetch and read state
   - ``contributionsRoutes.js`` — contribution calculations
   - ``noteRoutes.js`` — sticky-note widgets on the project board
   - ``fileRoutes.js`` — shared file upload and listing
   - ``aiRoutes.js`` — AI assistant chat

A complete list of endpoints with parameters and responses is in the
:doc:`/api-reference/index`.

Controllers
-----------

Controllers (in ``controllers/``) contain the per-endpoint logic:

- Validate the incoming request body and parameters
- Verify the user is authenticated and authorised for the action
- Call one or more model functions to read or write data
- Shape the response and send it back to the client
- Trigger notifications or socket broadcasts where appropriate

Controllers do not contain SQL. All database access is delegated to
the model layer.

Models
------

Models (in ``models/``) are the only layer that talks to the database
directly. Each model file exposes async functions that:

- Build parameterised SQL queries (using either the ``pg`` pool or
  the Supabase JS client depending on the operation)
- Execute the query and return the result
- Handle row-level edge cases (not found, empty results, etc.)

Keeping SQL in models means controllers stay clean and unit tests
can mock the data layer without spinning up a database.

Utils
-----

The ``utils/`` folder holds cross-cutting helpers that don't belong
to any single feature:

============================  ====================================================
File                          Purpose
============================  ====================================================
``auth.js``                   Passport.js Microsoft OAuth strategy and middleware
``session.js``                Express session configuration
``socket.js``                 Socket.io event handlers and room management
``supabase.js``               Supabase JS client + ``pg`` pool initialisation
``gemini.js``                 Google Gemini API client and prompt construction
``fileFetcher.js``            File download and Office text extraction
``emailSender.js``            Nodemailer transporter for notification emails
``emailConfig.js``            Mailtrap SMTP configuration
============================  ====================================================

Real-time updates
-----------------

GCMS uses Socket.io for real-time features. ``utils/socket.js`` sets
up the server-side socket and joins each authenticated client to a
room for every project they belong to.

When a relevant event occurs — a new chat message, a task status
change, a widget moved on the project board — the controller emits
a socket event to the project's room. All connected clients in that
project receive the update and refresh their UI without a page reload.

Socket events used:

- ``chat:message`` — new chat message
- ``task:update`` — task created, updated, or deleted
- ``widget:update`` — widget added, moved, or removed
- ``notification`` — new notification for a specific user

Authentication flow
-------------------

User authentication uses the OAuth 2.0 Authorization Code flow with
Microsoft as the identity provider, implemented via Passport.js.

.. mermaid::

   sequenceDiagram
       participant User
       participant GCMS
       participant Microsoft

       User->>GCMS: Click "Sign in with Microsoft"
       GCMS->>Microsoft: Redirect to /authorize
       User->>Microsoft: Enter credentials
       Microsoft->>GCMS: Redirect to /api/auth/callback with code
       GCMS->>Microsoft: Exchange code for access token
       Microsoft->>GCMS: Access token + refresh token
       GCMS->>GCMS: Create or update user record
       GCMS->>User: Redirect to dashboard with session cookie

The access token is stored in the session and used for subsequent
Microsoft Graph API calls (e.g. creating calendar events). The
refresh token is used to obtain a new access token when the current
one expires.

External integrations
---------------------

GCMS integrates with four external services. See
:doc:`/external-integrations/index` for full details.

- **Microsoft Graph API** — authentication and calendar sync
- **Google Gemini** — AI assistant
- **Supabase** — database and file storage
- **Mailtrap** — email notifications

Testing
-------

The ``Tests/`` folder contains Jest test suites organised by user
requirement (one file per UR, e.g. ``UR-1.test.js``). The
``Automated_Testing/`` folder contains standalone scripts for
integration-style tests that don't fit the unit-test pattern (e.g.
end-to-end notification flow).

See the project repository for instructions on running tests:

https://github.com/SETAP-Org/group-coursework-management-system