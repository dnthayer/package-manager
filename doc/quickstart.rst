.. _PyPI: https://pypi.python.org/pypi
.. _BroControl: https://www.zeek.org/sphinx/components/broctl/README.html

Quickstart Guide
================

Dependencies
------------

* Python 2.7+ or 3.0+
* git: https://git-scm.com
* GitPython: https://pypi.python.org/pypi/GitPython
* semantic_version: https://pypi.python.org/pypi/semantic_version
* btest: https://pypi.python.org/pypi/btest
* configparser backport (not needed when using Python 3.5+): https://pypi.python.org/pypi/configparser

Note that following the suggested `Installation`_ process via :program:`pip`
will automatically install dependencies for you.

Installation
------------

Using the latest stable release on PyPI_:

.. code-block:: console

  $ pip install bro-pkg

Using the latest git development version:

.. code-block:: console

  $ pip install git+git://github.com/bro/package-manager@master

Basic Configuration
-------------------

After installing via :program:`pip`, additional configuration is required.
First, make sure that the :program:`bro-config` script that gets installed with
:program:`bro` is in your :envvar:`PATH`.  Then, as the user you want to run
:program:`bro-pkg` with, do:

.. code-block:: console

  $ bro-pkg autoconfig

This automatically generates a config file with the following suggested
settings that should work for most Bro deployments:

- `script_dir`: set to the location of Bro's :file:`site` scripts directory
  (e.g. :file:`{<bro_install_prefix>}/share/bro/site`)

- `plugin_dir`: set to the location of Bro's default plugin directory (e.g.
  :file:`{<bro_install_prefix>}/lib/bro/plugins`)

- `bro_dist`: set to the location of Bro's source code.
  If you didn't build/install Bro from source code, this field will not be set,
  but it's only needed if you plan on installing packages that have uncompiled
  Bro plugins.

With those settings, the package manager will install Bro scripts, Bro plugins,
and BroControl plugins into directories where :program:`bro` and
:program:`broctl` will, by default, look for them.  BroControl clusters will
also automatically distribute installed package scripts/plugins to all nodes.

.. note::

  If your Bro installation is owned by "root" and you intend to run
  :program:`bro-pkg` as a different user, then you should grant "write" access
  to the directories specified by `script_dir` and `plugin_dir`.  E.g. you could
  do something like:

  .. code-block:: console

    $ sudo chgrp $USER $(bro-config --site_dir) $(bro-config --plugin_dir)
    $ sudo chmod g+rwX $(bro-config --site_dir) $(bro-config --plugin_dir)

The final step is to edit your :file:`site/local.bro`.  If you want to
have Bro automatically load the scripts from all
:ref:`installed <install-command>` packages that are also marked as
":ref:`loaded <load-command>`" add:

.. code-block:: bro

  @load packages

If you prefer to manually pick the package scripts to load, you may instead add
lines like :samp:`@load {<package_name>}`, where :samp:`{<package_name>}`
is the :ref:`shorthand name <package-shorthand-name>` of the desired package.

If you want to further customize your configuration, see the `Advanced
Configuration`_ section and also  check :ref:`here <bro-pkg-config-file>` for a
full explanation of config file options.  Otherwise you're ready to use
:ref:`bro-pkg <bro-pkg>`.

Advanced Configuration
----------------------

If you prefer to not use the suggested `Basic Configuration`_ settings for
`script_dir` and `plugin_dir`, the default configuration will install all
package scripts/plugins within :file:`~/.bro-pkg` or you may change them to
whatever location you prefer.  These will be referred to as "non-standard"
locations in the sense that vanilla configurations of either :program:`bro` or
:program:`broctl` will not detect scripts/plugins in those locations without
additional configuration.

When using non-standard location, follow these steps to integrate with
:program:`bro` and :program:`broctl`:

- To get command-line :program:`bro` to be aware of Bro scripts/plugins in a
  non-standard location, make sure the :program:`bro-config` script (that gets
  installed along with :program:`bro`) is in your :envvar:`PATH` and run:

  .. code-block:: console

    $ `bro-pkg env`

  Note that this sets up the environment only for the current shell session.

- To get :program:`broctl` to be aware of scripts/plugins in a non-standard
  location, run:

  .. code-block:: console

    $ bro-pkg config script_dir

  And set the `SitePolicyPath` option in :file:`broctl.cfg` based on the output
  you see.  Similarly, run:

  .. code-block:: console

    $ bro-pkg config plugin_dir

  And set the `SitePluginPath` option in :file:`broctl.cfg` based on the output
  you see.

Usage
-----

Check the output of :ref:`bro-pkg --help <bro-pkg>` for an explanation of all
available functionality of the command-line tool.

Package Upgrades/Versioning
~~~~~~~~~~~~~~~~~~~~~~~~~~~

When installing packages, note that the :ref:`install command
<install-command>`, has a ``--version`` flag that may be used to install
specific package versions which may either be git release tags or branch
names.  The way that :program:`bro-pkg` receives updates for a package
depends on whether the package is first installed to track stable
releases or a specific git branch.  See the :ref:`package upgrade
process <package-upgrade-process>` documentation to learn how
:program:`bro-pkg` treats each situation.

Offline Usage
~~~~~~~~~~~~~

It's common to have limited network/internet access on the systems where
Bro is deployed.  To accomodate those scenarios, :program:`bro-pkg` can
be used as normally on a system that *does* have network access to
create bundles of its package installation environment. Those bundles
can then be transferred to the deployment systems via whatever means are
appropriate (SSH, USB flash drive, etc).

For example, on the package management system you can do typical package
management tasks, like install and update packages:

.. code-block:: console

    $ bro-pkg install <package name>

Then, via the :ref:`bundle command <bundle-command>`, create a bundle
file which contains a snapshot of all currently installed packages:

.. code-block:: console

    $ bro-pkg bundle bro-packages.bundle

Then transfer :file:`bro-packages.bundle` to the Bro deployment
management host.  For Bro clusters using BroControl_, this will
be the system acting as the "manager" node.  Then on that system
(assuming it already as :program:`bro-pkg` installed and configured):

.. code-block:: console

    $ bro-pkg unbundle bro-packages.bundle

Finally, if you're using BroControl_, and the unbundling process
was successful, you need to deploy the changes to worker nodes:

.. code-block:: console

    $ broctl deploy
