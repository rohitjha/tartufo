
Usage
=====

History Scan
------------

By default, `tartufo` will scan the entire history of a git repo. The repo to
be scanned can be specified in one of two ways. The first, default behavior, is
by passing a git URL to `tartufo`. For example:

.. code-block:: sh

   $ tartufo https://github.com/godaddy/tartufo.git

When used this way, `tartufo` will clone the repository to a temporary
directory, scan the local clone, and then delete it.

Alternatively, if you already have a local clone, you can scan that directly
without the need for the temporary clone:

.. code-block:: sh

   $ tartufo --repo-path /path/to/my/repo

Pre-commit
----------

The ``--pre-commit`` flag instructs tartufo to scan staged, uncommitted changes
in a local repository. The repository location can be specified using
``--repo-path``, but it is legal to not supply a location; in this case, the
caller's current working directory is assumed to be somewhere within the local
clone's tree and the repository root is determined automatically.

The following example demonstrates how tartufo can be used to verify secrets
will not be committed to a git repository in error:

.. code-block:: sh
   :caption: .git/hooks/pre-commit

   #!/bin/sh

   # Redirect output to stderr.
   exec 1>&2

   # Check for suspicious content.
   tartufo --pre-commit --regex --entropy

Git will execute tartufo before committing any content. If problematic changes
are detected, they are reported by tartufo and git aborts the commit process.
Only when tartufo returns a success status (indicating no potential secrets
were discovered) will git commit the staged changes.

Note that it is always possible, although not recommended, to bypass the
pre-commit hook by using ``git commit --no-verify``.

If you would like to automate these hooks, you can use the `pre-commit`_ tool
by adding a ``.pre-commit-config.yaml`` file to your repository. You can copy
and paste the following to get you started:

.. code-block:: yaml

   - repo: https://github.com/godaddy/tartufo
     rev: master
     hooks:
     - id: tartufo

That's it! Now your contributors only need to run ``pre-commit install
--install-hooks``, and `tartufo` will automatically be run as a pre-commit hook.

.. warning::

   You probably don't actually want to use the `master` rev. This is the active
   development branch for this project, and can not be guaranteed stable. Your
   best bet would be to choose the latest version, currently |version|.

Temporary File Cleanup
----------------------

`tartufo` stores the results in temporary files, which are left on disk by
default, to allow inspection if problems are found. To automatically delete
these files when tartufo completes, specify the ``--cleanup`` flag:

.. code-block:: sh

   tartufo --cleanup
