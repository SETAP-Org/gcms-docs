rstContributing
============

Thank you for your interest in contributing to GCMS! This guide covers
everything you need to know to contribute effectively, whether you are
a member of the core team or an external contributor.

Getting Set Up
--------------

Before contributing, follow the :doc:`Getting Started <getting-started/index>`
guide to clone the repository, install dependencies, configure your environment,
and run the application locally.

External contributors should fork the repository on GitHub first, then clone
their fork instead of the main repository.

Branching Strategy
------------------

GCMS uses a feature branch workflow. The ``main`` branch is the source of
truth and is always kept in a working state.

**All work happens on feature branches.** Never commit directly to ``main``.

Create a new branch for each feature, fix, or change::

	git checkout main
	git pull origin main
	git checkout -b your-branch-name

Branch naming
~~~~~~~~~~~~~

Use short, descriptive branch names that reflect the purpose of the work.
Prefix branches by type:

- ``feature/`` for new features (e.g. ``feature/calendar-integration``)
- ``fix/`` for bug fixes (e.g. ``fix/login-redirect``)
- ``docs/`` for documentation changes (e.g. ``docs/api-reference``)
- ``refactor/`` for code restructuring (e.g. ``refactor/event-bus``)

Commit Messages
---------------

Write clear, concise commit messages that explain *what* changed and *why*.

**Good examples:**

- ``Add OneDrive file picker to project board``
- ``Fix session timeout on Microsoft auth callback``
- ``Refactor task controller to use event bus``

**Avoid:**

- ``update``, ``fix stuff``, ``changes``, ``wip``

Keep the first line under 72 characters. If more detail is needed, leave a
blank line and add a longer description below.

Pull Requests
-------------

When your work is ready, open a pull request against ``main``.

Before opening a PR
~~~~~~~~~~~~~~~~~~~

- Pull the latest ``main`` and resolve any merge conflicts on your branch
- Make sure the application still runs locally without errors
- Remove any debugging code, ``console.log`` statements, or commented-out blocks
- Verify no ``.env`` files or credentials are included in your commits

Opening a PR
~~~~~~~~~~~~

Your pull request should include:

- **A clear title** describing the change
- **A description** explaining what was changed and why
- **References to any related issues** (e.g. ``Closes #42``)
- **Screenshots or screen recordings** for any UI changes

Review process
~~~~~~~~~~~~~~

**All pull requests require at least one approval before merging.**

Reviewers will check that the code works as expected, follows existing
patterns in the codebase, and does not introduce regressions. Please
respond to review comments promptly and push fixes to the same branch —
do not open a new PR for revisions.

Once approved, the PR can be merged into ``main`` by the author or
a reviewer.

Code Style
----------

GCMS does not currently enforce a formal style guide, but contributions
should match the conventions already present in the codebase:

- Use existing folder structure — controllers in ``controllers/``,
  models in ``models/``, etc.
- Keep frontend logic in ``public/`` and templates in ``views/``
- Follow the existing event bus pattern for cross-module communication
- Match the indentation, quoting, and naming style of surrounding code

Reporting Issues
----------------

If you find a bug or have a feature suggestion, open an issue on GitHub:
https://github.com/SETAP-Org/group-coursework-management-system/issues

Include as much detail as possible:

- A clear title
- Steps to reproduce (for bugs)
- Expected vs actual behaviour
- Browser, OS, and any relevant environment details
- Screenshots if applicable

Security
--------

If you discover a security vulnerability, **do not open a public issue**.
Contact the maintainers directly so the issue can be addressed before
disclosure.

Questions
---------

If you are unsure about anything, open a draft PR or an issue to start
a discussion. We would rather answer questions early than have you spend
time on something that needs reworking later.
