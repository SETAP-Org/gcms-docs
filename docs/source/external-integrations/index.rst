External Integrations
=====================

GCMS integrates with external services to extend its functionality
beyond what the core application provides.

Microsoft Graph API
-------------------

GCMS uses the Microsoft Graph API for two purposes: user authentication
and calendar integration.

Authentication
~~~~~~~~~~~~~~

User login is handled through Microsoft OAuth via Passport.js. When a
user signs in with their Microsoft account, GCMS receives an access
token that is used for subsequent Graph API calls.

The OAuth flow requires an Azure App Registration with the following
configured:

- **Redirect URI** matching the URI used by the application
- **Client secret** for confidential client authentication
- **API permissions** for the scopes used by GCMS

Calendar Integration
~~~~~~~~~~~~~~~~~~~~

When a meeting is created on a project's calendar page, GCMS uses the
authenticated user's access token to create a corresponding event in
their personal Microsoft calendar.

Required scopes
~~~~~~~~~~~~~~~

The application requests the following scopes during the OAuth flow:

- ``openid`` — required for OpenID Connect authentication
- ``profile`` — basic profile information
- ``email`` — user's email address
- ``Calendars.ReadWrite`` — read and write access to the user's calendar

These are configured in the ``MICROSOFT_SCOPES`` environment variable
within ``.env.auth``.

Setting up your own Azure App Registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are setting up GCMS independently and need to register your own
Azure app:

1. Go to the `Azure Portal <https://portal.azure.com/>`_ and sign in
2. Navigate to **App Registrations** and click **New registration**
3. Set the redirect URI to ``http://localhost:3000/auth/microsoft/callback``
   for local development
4. Under **Certificates & Secrets**, generate a new client secret
5. Under **API Permissions**, add the scopes listed above
6. Copy the Application (client) ID and the secret value into your
   ``.env.auth`` file

.. note::

   For team members working on GCMS, the existing Azure App Registration
   credentials will be provided. You do not need to create your own.

Future integrations
-------------------

The following integrations are planned for future development.

OpenAI API
~~~~~~~~~~

OpenAI integration is on the GCMS roadmap. Planned use cases include
intelligent task suggestions, project summaries, and assistance with
contribution analysis. This feature is not yet implemented.
