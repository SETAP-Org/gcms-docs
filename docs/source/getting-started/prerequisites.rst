rstPrerequisites
=============

Before installing GCMS, make sure you have the following installed:

**Node.js**
   Required to run the Express.js server. Download from https://nodejs.org

**PostgreSQL 18**
   The database engine used by GCMS. Download from https://www.postgresql.org/download/

   After installation, verify PostgreSQL is running::

      sc query postgresql-x64-18

   If it is not running, start it with::

      net start postgresql-x64-18

**Git**
   Required to clone the repository. Download from https://git-scm.com

**Microsoft Account**
   GCMS uses Microsoft OAuth for authentication. You will need an Azure App Registration
   with a valid ``CLIENT_ID`` and ``CLIENT_SECRET`` to run the app locally.