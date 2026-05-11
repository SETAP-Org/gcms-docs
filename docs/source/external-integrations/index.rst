External Integrations
=====================

GCMS integrates with several external services to extend its functionality
beyond what the core application provides.

Microsoft Graph API
-------------------

GCMS uses the Microsoft Graph API for two purposes: user authentication
and calendar integration.

Authentication
~~~~~~~~~~~~~~

User login is handled through Microsoft OAuth via Passport.js using the
``passport-microsoft`` strategy. When a user signs in with their Microsoft
account, GCMS receives an access token and refresh token that are used
for subsequent Graph API calls.

The OAuth flow requires an Azure App Registration with the following
configured:

- **Redirect URI** matching the URI used by the application
  (``http://localhost:3000/api/auth/callback`` for local development)
- **Client secret** for confidential client authentication
- **API permissions** for the scopes used by GCMS

Required scopes
~~~~~~~~~~~~~~~

The application requests the following scopes during the OAuth flow:

- ``User.Read`` — basic user profile information
- ``Calendars.ReadWrite`` — read and write access to the user's calendar

Calendar integration
~~~~~~~~~~~~~~~~~~~~

When a meeting is created on a project's calendar page, GCMS uses the
authenticated user's access token to create a corresponding event in
their personal Microsoft calendar via the Graph API.

Setting up your own Azure App Registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are setting up GCMS independently and need to register your own
Azure app:

1. Go to the `Azure Portal <https://portal.azure.com/>`_ and sign in
2. Navigate to **App Registrations** and click **New registration**
3. Set the redirect URI to ``http://localhost:3000/api/auth/callback``
   for local development
4. Under **Certificates & Secrets**, generate a new client secret
5. Under **API Permissions**, add the scopes listed above
6. Copy the Application (client) ID and the secret value into your
   ``.env.auth`` file as ``CLIENT_ID`` and ``CLIENT_SECRET``

.. note::

   For team members working on GCMS, the existing Azure App Registration
   credentials will be provided. You do not need to create your own.

Google Gemini
-------------

GCMS uses the Google Gemini API to power the in-app AI assistant. The
assistant helps students understand their coursework specifications,
summarise documents, and answer questions about their project.

Model
~~~~~

The default model is ``gemini-2.5-flash``, chosen for its low cost and
fast response times. This is suitable for typical student Q&A and
document summarisation. The model can be swapped to ``gemini-2.5-pro``
for higher quality at higher cost.

File attachments
~~~~~~~~~~~~~~~~

When a user asks the assistant a question, GCMS can attach files from
the project's shared folder so the model can reason about their
contents. Files are handled in one of two ways depending on type:

**Sent natively to Gemini:**

- PDF (``application/pdf``)
- Plain text (``text/plain``)
- Markdown (``text/markdown``)
- CSV (``text/csv``)
- HTML (``text/html``)

**Extracted to plain text on the server before sending:**

- Word documents (``.docx``) — extracted with ``mammoth``
- Excel spreadsheets (``.xlsx``, ``.xls``) — extracted with ``xlsx``
- PowerPoint presentations (``.pptx``) — extracted with ``officeparser``

For Office formats, GCMS extracts the readable text content server-side
and forwards it to Gemini as ``text/plain``. This avoids the need for
native Office support in the model and keeps request sizes small.

A total budget of 15 MB is enforced across all attachments in a single
request. Files exceeding the budget or of unsupported types are
silently skipped, and the model is informed of any omissions so it
can reference this in its response.

System instruction
~~~~~~~~~~~~~~~~~~

Every Gemini request includes a system instruction that defines the
assistant's persona and behavioural guidelines. The assistant is
configured to:

- Keep responses concise
- Admit when it does not know an answer rather than inventing one
- Explain, summarise, and clarify rather than complete the coursework
  on the student's behalf

Configuration
~~~~~~~~~~~~~

The Gemini API key is stored in ``.env.gemini`` as ``GEMINI_API_KEY``.
The client is lazily initialised on the first request, so importing
``utils/gemini.js`` does not require the key to be set at startup.

Supabase
--------

GCMS uses Supabase for two distinct purposes: the primary PostgreSQL
database and shared file storage.

Database
~~~~~~~~

The PostgreSQL database is accessed via two clients depending on the
operation:

- The **Supabase JS client** (``@supabase/supabase-js``) is used for
  storage operations and any code that needs the Supabase API surface
- The **node-postgres** client (``pg``) is used for direct SQL queries
  via a connection pool

Both clients read connection details from the active environment file
(``.env.development`` or ``.env.production``).

Storage
~~~~~~~

Files uploaded to a project are stored in a Supabase Storage bucket.
The application stores only the file metadata
(``file_name``, ``storage_path``, ``size``, ``date_uploaded``) in the
``files`` table; the actual file contents live in the bucket.

The bucket name is configured via the
``SUPABASE_SHARED_FOLDERS_BUCKET`` environment variable.

Configuration
~~~~~~~~~~~~~

The Supabase client requires:

- ``SUPABASE_URL`` — the project URL
- ``SUPABASE_SERVICE_ROLE_KEY`` — the service role key for server-side access
- ``SUPABASE_SHARED_FOLDERS_BUCKET`` — the storage bucket name

These are read from ``.env.development`` or ``.env.production`` depending
on the current ``NODE_ENV``.

.. warning::

   The service role key bypasses all row-level security. It must never
   be exposed to the client or committed to version control.

Mailtrap
--------

GCMS uses `Mailtrap <https://mailtrap.io/>`_ as the SMTP provider for
sending notification emails to users.

How it works
~~~~~~~~~~~~

The application uses Nodemailer to send emails through Mailtrap's
SMTP service. When a notification is generated for a user who has
opted into email notifications (``users.email_notifications = true``),
a branded HTML email is sent with the notification content.

The email template includes the GCMS logo and styling, the notification
type and message, the related project name (if applicable), and a
prompt to log in to view more details.

Configuration
~~~~~~~~~~~~~

Mailtrap credentials are stored in ``.env.mailconfig``:

- ``MAILTRAP_HOST`` — SMTP host (defaults to ``sandbox.smtp.mailtrap.io``)
- ``MAILTRAP_PORT`` — SMTP port (defaults to ``2525``)
- ``MAILTRAP_USER`` — SMTP username
- ``MAILTRAP_PASS`` — SMTP password
- ``EMAIL_FROM`` — sender address (defaults to ``noreply@gcms.app``)

The transporter is initialised on application start and verifies the
SMTP connection on boot.

.. note::

   GCMS currently uses Mailtrap's sandbox environment, which captures
   outgoing emails for inspection rather than actually delivering them.
   For production use, the SMTP configuration would be switched to a
   transactional email provider with real delivery.