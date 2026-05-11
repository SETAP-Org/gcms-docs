Entity Reference
================

Detailed reference for every table in the GCMS database.

users
-----

Stores user account information populated from Microsoft OAuth on
first sign-in.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``user_id``                   ``uuid``               Primary key, auto-generated
``user_first_name``           ``varchar``            From Microsoft profile
``user_last_name``            ``varchar``            From Microsoft profile
``user_email``                ``varchar``            Unique, from Microsoft profile
``microsoft_id``              ``text``               Unique Microsoft account identifier
``username``                  ``varchar``            Unique, 3–20 characters, user-chosen
``date_created``              ``timestamptz``        Account creation timestamp
``last_login``                ``timestamptz``        Most recent login timestamp
``email_notifications``       ``boolean``            Whether the user has opted into email notifications, default ``false``
============================  =====================  =============================================

projects
--------

Represents a group coursework project. Each project has one team leader
and a deadline, with members joined via ``user_projects``.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``project_id``                ``uuid``               Primary key, auto-generated
``team_leader_id``            ``uuid``               FK → ``users.user_id``
``created_by``                ``uuid``               FK → ``users.user_id``
``project_name``              ``varchar``            Display name
``project_deadline``          ``date``               Final due date
``p_date_created``            ``timestamptz``        Creation timestamp
``p_time_updated``            ``timestamptz``        Last update timestamp
============================  =====================  =============================================

user_projects
-------------

Junction table establishing the many-to-many relationship between
users and projects.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``user_id``                   ``uuid``               FK → ``users.user_id``
``project_id``                ``uuid``               FK → ``projects.project_id``
============================  =====================  =============================================

The composite primary key ``(user_id, project_id)`` ensures a user
cannot be added to the same project twice.

tasks
-----

Individual units of work within a project. Tasks have weights that
feed into the contribution tracking system.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``task_id``                   ``uuid``               Primary key, auto-generated
``project_id``                ``uuid``               FK → ``projects.project_id``
``assignee_id``               ``uuid``               FK → ``users.user_id``, nullable
``task_title``                ``varchar``            Short title
``task_description``          ``text``               Optional detail
``task_weight``               ``numeric``            Used to calculate contribution percentages
``task_status``               ``task_status``        Enum (see Custom Types below)
``task_deadline``             ``date``               Optional task-level deadline
``t_date_created``            ``timestamptz``        Creation timestamp
``t_time_updated``            ``timestamptz``        Last update timestamp
============================  =====================  =============================================

messages
--------

Chat messages within a project's group chat. Loaded historically
on page open and updated in real-time via Socket.io.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``message_id``                ``uuid``               Primary key, auto-generated
``sender_id``                 ``uuid``               FK → ``users.user_id``
``project_id``                ``uuid``               FK → ``projects.project_id``
``message_content``           ``text``               Message body
``m_date_sent``               ``timestamptz``        Send timestamp
============================  =====================  =============================================

meetings
--------

Scheduled meetings within a project. Each meeting also creates a
corresponding event in the team leader's Microsoft calendar via
the Graph API.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``meeting_id``                ``uuid``               Primary key, auto-generated
``team_leader_id``            ``uuid``               FK → ``users.user_id``
``project_id``                ``uuid``               FK → ``projects.project_id``
``scheduled_time``            ``timestamptz``        When the meeting starts
``meeting_duration``          ``integer``            Duration in minutes
``meeting_location``          ``meeting_location``   Enum (see Custom Types below)
``meeting_notes``             ``text``               Optional agenda or notes
============================  =====================  =============================================

meeting_attendances
-------------------

Tracks which users attended which meetings.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``attendance_id``             ``uuid``               Primary key, auto-generated
``user_id``                   ``uuid``               FK → ``users.user_id``
``meeting_id``                ``uuid``               FK → ``meetings.meeting_id``
``attendance_status``         ``attendance_status``  Enum (see Custom Types below)
``check_in_time``             ``timestamptz``        Time the user checked in
============================  =====================  =============================================

notifications
-------------

Per-user notifications generated by activity across projects.
Includes denormalised ``target_username`` and ``project_name`` fields
to avoid extra joins when rendering notification messages.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``notification_id``           ``uuid``               Primary key, auto-generated
``user_id``                   ``uuid``               FK → ``users.user_id``, recipient
``project_id``                ``uuid``               FK → ``projects.project_id``, nullable
``notification_type``         ``notification_type``  Enum (see Custom Types below)
``notification_message``      ``text``               Display text
``n_date_created``            ``timestamptz``        Creation timestamp
``target_username``           ``text``               Denormalised — user the notification refers to
``project_name``              ``text``               Denormalised — project the notification refers to
============================  =====================  =============================================

widgets
-------

Sticky-note style widgets placed on the interactive project board.
Each widget has a position on the board (``widget_x``, ``widget_y``)
and free-form text.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``widget_id``                 ``uuid``               Primary key, auto-generated
``project_id``                ``uuid``               FK → ``projects.project_id``
``widget_x``                  ``numeric``            X coordinate on the board
``widget_y``                  ``numeric``            Y coordinate on the board
``widget_text``               ``varchar``            Widget content, optional
============================  =====================  =============================================

files
-----

Metadata for files uploaded to a project's shared folder. The actual file
contents are stored in a Supabase Storage bucket; this table only tracks
the metadata.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``file_id``                   ``uuid``               Primary key, auto-generated
``project_id``                ``uuid``               FK → ``projects.project_id``
``file_name``                 ``varchar``            Original file name as uploaded
``storage_path``              ``text``               Path within the Supabase Storage bucket
``size``                      ``bigint``             File size in bytes
``date_uploaded``             ``timestamp``          Upload timestamp
============================  =====================  =============================================

ai_chat_messages
----------------

Stores the per-project chat history with the GCMS AI assistant
(powered by Google Gemini). Each row represents one message in the
conversation, from either the user or the assistant.

============================  =====================  =============================================
Column                        Type                   Notes
============================  =====================  =============================================
``ai_message_id``             ``uuid``               Primary key, auto-generated
``project_id``                ``uuid``               FK → ``projects.project_id``, ``ON DELETE CASCADE``
``user_id``                   ``uuid``               FK → ``users.user_id``, ``ON DELETE SET NULL``
``role``                      ``ai_chat_role``       Enum (see Custom Types below)
``content``                   ``text``               Message body
``ai_date_sent``              ``timestamptz``        Send timestamp, defaults to ``NOW()``
============================  =====================  =============================================

An index on ``(project_id, ai_date_sent)`` keeps history retrieval fast
even as conversations grow.

Custom Types
------------

GCMS uses several PostgreSQL enum types to constrain values to a
fixed set of options.

``task_status``
   Used by ``tasks.task_status``. Represents the lifecycle state of a task.

   - ``To Do``
   - ``In Progress``
   - ``Completed``

``meeting_location``
   Used by ``meetings.meeting_location``. Indicates whether a meeting
   takes place online or in person.

   - ``Virtual``
   - ``Presential``

``attendance_status``
   Used by ``meeting_attendances.attendance_status``. Records whether
   a user attended a meeting.

   - ``Present``
   - ``Not Present``

``notification_type``
   Used by ``notifications.notification_type``. Determines the category
   of a notification, used by the frontend to choose icon and styling.

   - ``Message`` — a new chat message
   - ``Project`` — project-level update
   - ``Member Leave`` — a member left the project
   - ``Member Join`` — a member joined the project
   - ``Leader`` — leadership change
   - ``Task`` — task assigned, updated, or completed

``ai_chat_role``
   Used by ``ai_chat_messages.role``. Distinguishes who sent each
   message in an AI assistant conversation.

   - ``user`` — the human's message
   - ``assistant`` — the AI's response