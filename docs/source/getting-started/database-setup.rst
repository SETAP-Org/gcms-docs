Database Setup
==============

GCMS uses Supabase as its primary database, with PostgreSQL as an
optional local alternative. The setup process is the same for both:
apply the schema, optionally seed development data.

Using Supabase (default)
------------------------

If you are using the shared Supabase credentials provided to team
members, the database is already set up — you do not need to do
anything. Skip ahead to :doc:`running-the-app`.

If you are running against your own Supabase project, you will need
to apply the schema once before starting the app. Run::

   npm run install_db

This executes ``db/schema.sql`` followed by ``db/seed_dev.sql`` against
the database configured in your active ``.env`` file
(``.env.development`` for ``npm run dev``, ``.env.production`` for
``npm run prod``).

You will also need to create a Storage bucket in your Supabase project
and set ``SUPABASE_SHARED_FOLDERS_BUCKET`` to its name in your env file.

Using local PostgreSQL
----------------------

If you are using a local PostgreSQL instance instead of Supabase,
edit your ``.env.development`` to point at the local instance::

   DB_URL=postgresql://[username]:[password]@localhost:5432/gcms

Then run::

   npm run install_db

This creates all required tables in your local ``gcms`` database.

.. note::

   Local PostgreSQL setup does not provide a storage equivalent of
   Supabase Storage. File upload features will not work without a
   Supabase bucket configured.

Schema reference
----------------

The full schema is defined in ``db/schema.sql``. For a documented
reference of every table, column, and enum, see
:doc:`/database-schema/entity-reference`.

For an entity relationship diagram, see
:doc:`/database-schema/er-diagram`.

Troubleshooting
---------------

If ``npm run install_db`` fails, see the :doc:`/troubleshooting` page
for common issues including PostgreSQL service problems and TCP/IP
configuration on Windows.