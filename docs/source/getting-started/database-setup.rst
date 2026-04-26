rstDatabase Setup
==============

GCMS includes an automated database setup script. Run it once after configuring your ``.env``::

   npm run install_db

This will create all required tables in your PostgreSQL ``gcms`` database.

Troubleshooting
---------------

If the script fails, first check that PostgreSQL is running::

   sc query postgresql-x64-18

If the port is not responding, verify your ``postgresql.conf`` has TCP/IP enabled.
Open ``C:\Program Files\PostgreSQL\18\data\postgresql.conf`` and ensure::

   listen_addresses = '*'
   port = 5432

Then open ``C:\Program Files\PostgreSQL\18\data\pg_hba.conf`` and ensure::

   host all all 127.0.0.1/32 trust
   host all all ::1/128 trust

Restart PostgreSQL after any config changes::

   net stop postgresql-x64-18
   net start postgresql-x64-18