Prerequisites
=============

Before installing GCMS, make sure you have the following installed and
configured:

Required
--------

**Node.js**
   Required to run the Express.js server. Download from https://nodejs.org

**Git**
   Required to clone the repository. Download from https://git-scm.com

**Microsoft Account**
   GCMS uses Microsoft OAuth for authentication. You will need access
   to a valid Azure App Registration to run the app locally. The
   existing app registration credentials will be provided to team
   members — see :doc:`environment-configuration`.

Database
--------

GCMS uses Supabase as its primary database. Team members are provided
with shared Supabase credentials and do not need to install PostgreSQL
locally.

If you want to develop offline or run GCMS against your own database,
you can install PostgreSQL locally and use it instead:

**PostgreSQL 18 (optional, for local development)**
   Download from https://www.postgresql.org/download/

   After installation, verify PostgreSQL is running::

      sc query postgresql-x64-18

   If it is not running, start it with::

      net start postgresql-x64-18

   You will need to manually configure the ``DB_URL`` in your
   ``.env.development`` to point at your local instance instead of
   the shared Supabase database, and apply the schema yourself
   (see :doc:`database-setup`).

External services
-----------------

You will need credentials for the following external services. Team
members are provided these — see :doc:`environment-configuration` for
details:

- Microsoft Azure App Registration (OAuth + Graph API)
- Supabase project (database + storage)
- Google Gemini (AI assistant)
- Mailtrap or another SMTP provider (email notifications)