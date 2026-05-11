Environment Configuration
=========================

GCMS uses multiple ``.env`` files to separate concerns. These files are not
committed to version control and must be obtained from an existing team member
before running the application.

Required files
--------------

Place all six files in the **project root** alongside ``server.js``:

``.env.development``
   Supabase connection string and storage configuration for the
   development database. Used when running ``npm run dev``.

``.env.production``
   Supabase connection string and storage configuration for the
   production database. Used when running ``npm run prod``.

``.env.auth``
   Microsoft Azure OAuth credentials, including ``CLIENT_ID`` and
   ``CLIENT_SECRET``.

``.env.session-secret``
   A shared session secret key used by Express to sign session cookies.

``.env.gemini``
   Google Gemini API credentials for the in-app AI assistant.

``.env.mailconfig``
   Mailtrap SMTP credentials used by Nodemailer to send notification
   emails.

Expected variables
------------------

The following variables are read from each file:

``.env.development`` / ``.env.production``
   - ``DB_URL`` — full PostgreSQL connection string for the
     ``node-postgres`` pool
   - ``SUPABASE_URL`` — Supabase project URL
   - ``SUPABASE_SERVICE_ROLE_KEY`` — Supabase service role key, used
     server-side for storage operations
   - ``SUPABASE_SHARED_FOLDERS_BUCKET`` — name of the Supabase Storage
     bucket used for project file uploads

``.env.auth``
   - ``CLIENT_ID`` — Azure App Registration client ID
   - ``CLIENT_SECRET`` — Azure App Registration client secret

``.env.session-secret``
   - ``SESSION_SECRET`` — any long random string

``.env.gemini``
   - ``GEMINI_API_KEY`` — Google AI Studio API key

``.env.mailconfig``
   - ``MAILTRAP_HOST`` — SMTP host
   - ``MAILTRAP_PORT`` — SMTP port
   - ``MAILTRAP_USER`` — SMTP username
   - ``MAILTRAP_PASS`` — SMTP password
   - ``EMAIL_FROM`` — optional sender address

Obtaining credentials
---------------------

All developers working on GCMS share the same set of environment files.
Contact an existing team member to receive them before attempting to
run the application locally.

If you are setting up GCMS independently, you will need to provision
your own resources:

- A Supabase project, with a Storage bucket created and the schema
  applied (see :doc:`database-setup`)
- An Azure App Registration with the required scopes
  (see :doc:`/external-integrations/index`)
- A Google AI Studio API key for Gemini access
- A Mailtrap account or another SMTP provider

.. warning::

   Never commit any ``.env`` file to version control. All env files
   are covered by ``.gitignore``. The Supabase service role key in
   particular bypasses all row-level security and must be kept secret.