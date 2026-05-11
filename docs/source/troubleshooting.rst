Troubleshooting
===============

Common issues encountered when setting up or running GCMS, and how to
resolve them.

Supabase connection issues
--------------------------

Database connection refused
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the app fails to connect to Supabase, check the following:

- ``DB_URL`` in your active env file is the full PostgreSQL connection
  string from the Supabase dashboard, not the Supabase project URL
- ``SUPABASE_URL`` and ``SUPABASE_SERVICE_ROLE_KEY`` are also set —
  the storage client needs these in addition to ``DB_URL``
- The active env file matches your ``NODE_ENV`` — ``npm run dev``
  reads ``.env.development``, ``npm run prod`` reads ``.env.production``

Service role key rejected
~~~~~~~~~~~~~~~~~~~~~~~~~

If file storage operations fail with an authorization error, your
``SUPABASE_SERVICE_ROLE_KEY`` may be invalid or expired. Regenerate
it from the Supabase dashboard under
**Project Settings → API → Service role secret**.

.. warning::

   The service role key bypasses row-level security. Never expose
   it client-side or commit it to version control.

Storage bucket not found
~~~~~~~~~~~~~~~~~~~~~~~~

If file uploads fail with a bucket-related error, make sure:

- ``SUPABASE_SHARED_FOLDERS_BUCKET`` in your env file matches the
  exact name of the bucket in Supabase
- The bucket exists in the same Supabase project as your database

Local PostgreSQL
----------------

This section only applies if you are running GCMS against a local
PostgreSQL instance instead of Supabase.

Checking if PostgreSQL is running
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Windows, open Command Prompt or PowerShell and run::

   sc query postgresql-x64-18

Look for ``STATE : 4 RUNNING`` in the output. If it shows ``STOPPED``,
start the service::

   net start postgresql-x64-18

Verifying the port is open
~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, PostgreSQL listens on port ``5432``. Check that something
is listening on it::

   netstat -ano | findstr :5432

If nothing returns, the service is either stopped or the port has been
changed in the config.

Connection refused or TCP/IP errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the application or ``psql`` cannot connect to the database, TCP/IP
may be disabled or the listen address may be misconfigured.

Open the configuration file (you may need administrator privileges)::

   C:\Program Files\PostgreSQL\18\data\postgresql.conf

Uncomment and update these lines::

   listen_addresses = '*'
   port = 5432

Then open::

   C:\Program Files\PostgreSQL\18\data\pg_hba.conf

Add or update these lines to allow local connections (development only):

.. code-block:: text

   host all all 127.0.0.1/32 trust
   host all all ::1/128 trust

.. warning::

   The ``trust`` authentication method should only be used for local
   development. Never use it in production environments.

Restart PostgreSQL after any config changes::

   net stop postgresql-x64-18
   net start postgresql-x64-18

Database setup script fails
---------------------------

If ``npm run install_db`` fails, work through this checklist:

- Your active env file exists and contains a valid ``DB_URL``
- The database referenced in your connection string exists
- The database user has permission to create tables, types, and
  extensions (Supabase service role accounts have this by default)

Node.js issues
--------------

Module not found errors
~~~~~~~~~~~~~~~~~~~~~~~

If you see ``Cannot find module`` errors after pulling new changes,
dependencies may be out of date. Run::

   npm install

Port already in use
~~~~~~~~~~~~~~~~~~~

If the server fails to start with ``EADDRINUSE``, another process is
using port 3000. Find and stop it::

   netstat -ano | findstr :3000

Take note of the PID in the last column, then::

   taskkill /PID <pid> /F

Microsoft authentication
------------------------

Redirect URI mismatch
~~~~~~~~~~~~~~~~~~~~~

If Microsoft login fails with a redirect URI error, the URI registered
in your Azure App Registration must exactly match the one used by the
application. The default for local development is
``http://localhost:3000/api/auth/callback``. Check the Azure portal
under **App Registrations → your app → Authentication**.

Invalid client secret
~~~~~~~~~~~~~~~~~~~~~

Client secrets in Azure expire after a set period. If authentication
suddenly stops working, generate a new secret in the Azure portal and
update ``.env.auth``.

AI assistant
------------

Gemini API key not set
~~~~~~~~~~~~~~~~~~~~~~

If the AI assistant returns an error mentioning missing credentials,
``GEMINI_API_KEY`` is not set or not loaded. Verify that
``.env.gemini`` exists in the project root and contains a valid key
from `Google AI Studio <https://aistudio.google.com/app/apikey>`_.

Office files not being processed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If Word, Excel, or PowerPoint files aren't being included in AI
responses, check the server logs for parsing errors. The server-side
extraction libraries (``mammoth``, ``xlsx``, ``officeparser``) require
the files to be valid and readable. Corrupted or password-protected
files will be skipped.

Email notifications
-------------------

Emails not being sent
~~~~~~~~~~~~~~~~~~~~~

If email notifications are not being delivered:

- Check ``.env.mailconfig`` is present with valid Mailtrap credentials
- Verify the recipient user has ``email_notifications = true`` in the
  ``users`` table
- Check the server console at startup — the transporter logs
  ``email transporter ready`` on success or an authentication error
  on failure

.. note::

   GCMS uses Mailtrap's sandbox environment by default. Emails are
   captured for inspection in the Mailtrap dashboard rather than
   being delivered to real inboxes.

Still stuck?
------------

If none of the above resolves your issue, check the GitHub issue
tracker for similar reports, or open a new issue with full details
of the problem and any error messages:

https://github.com/SETAP-Org/group-coursework-management-system/issues