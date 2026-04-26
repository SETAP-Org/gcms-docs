Environment Configuration
=========================

GCMS uses multiple ``.env`` files to separate concerns. These files are not
committed to version control and must be obtained from an existing team member
before running the application.

Required files
--------------

Place all four files in the **project root** alongside ``server.js``:

``.env.development``
	Supabase connection string for the development database.

``.env.production``
	Supabase connection string for the production database.

``.env.auth``
	Microsoft Azure OAuth credentials, including ``CLIENT_ID``,
	``CLIENT_SECRET``, and ``MICROSOFT_SCOPES``.

``.env.session-secret``
	A shared session secret key used by Express to sign session cookies.

Obtaining credentials
---------------------

All developers currently share the same set of environment files.
Contact an existing team member to receive them before attempting to
run the application locally.

.. warning::

	Never commit any ``.env`` file to version control. All env files
	are covered by ``.gitignore``.