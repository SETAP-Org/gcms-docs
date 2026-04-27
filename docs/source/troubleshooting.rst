rstTroubleshooting
===============

Common issues encountered when setting up or running GCMS, and how to
resolve them.

PostgreSQL
----------

Most setup issues with GCMS are related to PostgreSQL configuration
on Windows. The steps below assume PostgreSQL 18 installed in the
default location.

Checking if PostgreSQL is running
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open Command Prompt or PowerShell and run::

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

Checking the service configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To inspect the full service configuration::

	sc qc postgresql-x64-18

This shows the binary path, startup type, and dependencies — useful
for diagnosing service-level problems.

Connection refused or TCP/IP errors
-----------------------------------

If the application or ``psql`` cannot connect to the database, TCP/IP
may be disabled or the listen address may be misconfigured.

Step 1 — Edit ``postgresql.conf``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open the configuration file (you may need administrator privileges)::

	C:\Program Files\PostgreSQL\18\data\postgresql.conf

Find these lines:

.. code-block:: text

	#listen_addresses = 'localhost'
	#port = 5432

Uncomment them and change the listen address::

	listen_addresses = '*'
	port = 5432

Step 2 — Edit ``pg_hba.conf``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Open::

	C:\Program Files\PostgreSQL\18\data\pg_hba.conf

Add or update these lines to allow local connections without password
authentication (development only):

.. code-block:: text

	host all all 127.0.0.1/32 trust
	host all all ::1/128 trust

.. warning::

	The ``trust`` authentication method should only be used for local
	development. Never use it in production environments.

Step 3 — Restart the service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration changes only take effect after a restart::

	net stop postgresql-x64-18
	net start postgresql-x64-18

Database setup script fails
---------------------------

If ``npm run install_db`` fails, work through this checklist:

- PostgreSQL service is running (see above)
- Your ``.env.development`` file exists and contains a valid ``DB_URL``
- The database name in your connection string exists in PostgreSQL
- Your PostgreSQL user has permission to create tables

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
application. Check the Azure portal under
**App Registrations → your app → Authentication**.

Invalid client secret
~~~~~~~~~~~~~~~~~~~~~

Client secrets in Azure expire after a set period. If authentication
suddenly stops working, generate a new secret in the Azure portal and
update ``.env.auth``.

Still stuck?
------------

If none of the above resolves your issue, check the GitHub issue tracker
for similar reports, or open a new issue with full details of the
problem and any error messages you are seeing:

https://github.com/SETAP-Org/group-coursework-management-system/issues
