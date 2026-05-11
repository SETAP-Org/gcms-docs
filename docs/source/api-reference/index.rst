API Reference
=============

GCMS exposes two types of HTTP endpoints:

- **Page routes** — server-side rendered EJS views, mounted at ``/``
- **API routes** — JSON endpoints used by client-side JavaScript,
  mounted at ``/api``

This page documents the API routes. For the architectural context of
how routes fit into the request flow, see :doc:`/architecture/index`.

Conventions
-----------

- All API routes are prefixed with ``/api``
- Most routes require authentication via a session cookie set by the
  Microsoft OAuth flow
- Project-scoped routes apply ``checkMembership`` middleware to verify
  the authenticated user is a member of the project
- Request and response bodies are JSON unless otherwise noted
- ``:param`` syntax denotes URL parameters
- IDs are UUIDs unless otherwise noted

Common responses
~~~~~~~~~~~~~~~~

============  ====================================================
Status        Meaning
============  ====================================================
``200 OK``    Successful GET, PUT, or DELETE
``201``       Successful POST (resource created)
``302``       Redirect (typically after login or sign-out)
``400``       Bad request — missing or invalid parameters
``401``       Unauthenticated
``403``       Authenticated but not authorised for this resource
``404``       Resource not found
``500``       Server error
============  ====================================================

Authentication
--------------

OAuth flow endpoints for signing in with Microsoft.

``GET /api/auth``
   Initiates the Microsoft OAuth flow. If the user is already logged
   in, redirects them. Otherwise, redirects to Microsoft's consent page.

``GET /api/auth/callback``
   OAuth callback URL. Microsoft redirects here after the user
   authenticates. Sets the session cookie and the "just authenticated"
   flag, then redirects to the welcome page.

``GET /api/auth/signout``
   Clears the session and redirects the user to the login page.

``GET /api/auth/justAuthenticated``
   Returns the "just authenticated" flag, used by the welcome page
   to show a one-time welcome message.

Users
-----

Endpoints for the current user's account.

``GET /api/me``
   Returns the authenticated user's profile information from the
   database.

``GET /api/me/photo``
   Returns the authenticated user's profile photo from Microsoft
   Graph (binary image data). Requires a valid Microsoft access token
   in the session.

``POST /api/users/addUser``
   Creates a new user record after the first OAuth login. Populated
   from the Microsoft profile in the session.

``PUT /api/users/changeUsername``
   Updates the authenticated user's chosen username. Validates that
   the new username is 3–20 characters and unique.

   **Body:** ``{ "username": "newName" }``

Projects
--------

Endpoints for creating, reading, updating, and deleting projects.

``GET /api/me/projects``
   Returns all projects the authenticated user is a member of.

``GET /api/projects/:project_id``
   Returns the details of a single project, including members and
   team leader.

``POST /api/projects/addProject``
   Creates a new project. The authenticated user becomes the team
   leader and is automatically added as a member.

   **Body:**

   .. code-block:: json

      {
        "project_name": "My Project",
        "project_deadline": "2026-06-01"
      }

``POST /api/projects/user``
   Adds a user to an existing project. Only the team leader can
   invite members.

   **Body:** ``{ "project_id": "uuid", "user_id": "uuid" }``

``PUT /api/projects/leader``
   Transfers team leadership to another member of the project. Only
   the current team leader can perform this action.

   **Body:** ``{ "project_id": "uuid", "new_leader_id": "uuid" }``

``DELETE /api/projects/user``
   Removes a user from a project. The team leader can remove anyone;
   a member can remove themselves.

   **Body:** ``{ "project_id": "uuid", "user_id": "uuid" }``

``DELETE /api/projects/:project_id``
   Deletes the project and all related data (tasks, messages,
   meetings, etc.). Only the team leader can perform this action.

Tasks
-----

Endpoints for managing tasks within a project. All routes are
project-scoped and apply ``checkMembership`` where indicated.

``GET /api/projects/:project_id/tasks``
   Returns all tasks for the specified project.

``POST /api/tasks/:project_id/addTask``
   Creates a new task within the project.

   **Body:**

   .. code-block:: json

      {
        "task_title": "Write report",
        "task_description": "Optional details",
        "task_weight": 10,
        "task_deadline": "2026-05-01",
        "assignee_id": "uuid or null"
      }

``PUT /api/projects/:project_id/tasks/:task_id/updateStatus``
   Updates the status of a task. Membership is verified.

   **Body:** ``{ "task_status": "To Do | In Progress | Completed" }``

``DELETE /api/projects/:project_id/tasks/:task_id``
   Deletes a task. Membership is verified.

Calendar
--------

Endpoints for project meetings, synchronised with Microsoft Calendar
via the Graph API. All routes require a valid Microsoft access token
in the session.

``GET /api/calendar/events``
   Returns events from the authenticated user's Microsoft calendar.

``POST /api/calendar/events``
   Creates a new event in the user's Microsoft calendar and stores
   a corresponding meeting in the GCMS database.

   **Body:**

   .. code-block:: json

      {
        "project_id": "uuid",
        "scheduled_time": "2026-05-01T10:00:00Z",
        "meeting_duration": 60,
        "meeting_location": "Virtual | Presential",
        "meeting_notes": "Optional"
      }

``DELETE /api/calendar/events/:eventId``
   Deletes the specified event from both Microsoft Calendar and the
   GCMS database.

Notifications
-------------

Endpoints for retrieving and dismissing user notifications.

``GET /api/notifications/:user_id``
   Returns all notifications for the specified user, ordered by date
   created (most recent first).

``DELETE /api/notifications/:notification_id``
   Deletes a single notification.

Contributions
-------------

Endpoint for calculating project contributions.

``GET /api/contributions/:project_id``
   Returns the contribution breakdown for the project, calculated
   from completed tasks and their weights. Returns each member's
   percentage of total completed work.

Notes / Widgets
---------------

Endpoints for the sticky-note widgets on the project board, rendered
with Konva.js.

``POST /api/notes``
   Adds a new widget to a project board.

   **Body:**

   .. code-block:: json

      {
        "project_id": "uuid",
        "widget_x": 100,
        "widget_y": 200,
        "widget_text": "Optional"
      }

``PUT /api/notes/:note_id``
   Updates an existing widget's position or text.

   **Body:** ``{ "widget_x": 150, "widget_y": 250, "widget_text": "..." }``

``DELETE /api/notes/:note_id``
   Removes a widget from the project board.

Files
-----

Endpoints for uploading, listing, downloading, and deleting files in
a project's shared Supabase Storage bucket.

``POST /api/projects/:project_id/files/upload-init``
   Generates a signed upload URL for direct browser-to-Supabase
   upload. The client uploads to the returned URL, then calls
   ``metadata`` below to record the file.

``POST /api/projects/:project_id/files/metadata``
   Records metadata for a file that was just uploaded.

   **Body:** ``{ "file_name": "...", "storage_path": "...", "size": 12345 }``

``GET /api/projects/:project_id/files/metadata``
   Returns metadata for all files in the project's shared folder.

``GET /api/projects/:project_id/files/download``
   Returns a signed download URL for the requested file.

   **Query:** ``?file_id=uuid``

``DELETE /api/projects/:project_id/files/:file_id``
   Deletes a file from both Supabase Storage and the database.

AI Assistant
------------

Endpoints for the per-project AI chat powered by Google Gemini.
All routes apply ``checkMembership``.

``GET /api/projects/:project_id/ai-chat``
   Returns the chat history for the project, ordered chronologically.

``POST /api/projects/:project_id/ai-chat``
   Sends a message to the AI assistant. The request may include a list
   of file IDs to attach as context. Files are downloaded from Supabase
   Storage and (for Office formats) extracted to text before being
   forwarded to Gemini.

   **Body:**

   .. code-block:: json

      {
        "message": "Summarise the spec",
        "file_ids": ["uuid", "uuid"]
      }

   **Response:** The full updated chat history including the assistant's reply.

``DELETE /api/projects/:project_id/ai-chat``
   Clears all chat history for the project. Useful when the user
   wants to start a fresh conversation.

Page routes
-----------

For reference, the server-side rendered pages mounted at ``/`` are:

============================================================  ========================
Route                                                         View
============================================================  ========================
``/``                                                         Landing or redirect
``/welcome``                                                  First-login welcome
``/error``                                                    Generic error page
``/:username``                                                User dashboard
``/:username/projects``                                       Projects list
``/:username/profile``                                        User profile
``/:username/projects/:project_id``                           Project dashboard
``/:username/projects/:project_id/information``               Project info / settings
``/:username/projects/:project_id/tasks``                     Tasks page
``/:username/projects/:project_id/calendar``                  Calendar page
``/:username/projects/:project_id/chat``                      Group chat
``/:username/projects/:project_id/contributions``             Contributions page
``/:username/projects/:project_id/files``                     Shared files
============================================================  ========================

All project-scoped page routes apply ``checkMembership`` middleware.