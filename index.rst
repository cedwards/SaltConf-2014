===============
LAMP Deployment
===============

.. figure:: /_static/saltconf.jpg

Introduction
------------

 - Christer Edwards (cedwards)
 - Package Maintainer: FreeBSD, Arch Linux
 - http://intothesaltmine.org
 - Local to SLC

.. note::

    Try to make a bit of conversation here.

    Are there any FreeBSD users in here?
    Are there any Arch users in here?

    How many people are from out of town? Where?

Overview
--------

This presentation is broken up into a few parts. There is a lecture /
discussion portion and lab portions. We'll discuss a couple of different
methods for deploying a LAMP stack using Salt, and then implement what we
discussed.

.. note::

    How many people are new-ish to SaltStack?
    
    How many people are new-ish to writing States?

    How many people have used Formulas?

Requirements
------------

This lab requires a running Salt minion and master. Preferrably a test system;
perhaps a virtual machine. 

The demonstrations in this lab will be done on CentOS 6.5 and Salt 0.17.4.

Salt States
-----------

Using Salt States to define the packages and services we want to run.

The Salt State tree:
--------------------

.. code-block:: bash

    /srv/salt/
    ├── apache.sls
    ├── mysql.sls
    ├── php.sls
    └── top.sls

OR

.. code-block:: bash

    /srv/salt/
    ├── apache
    │   └── init.sls
    ├── mysql
    │   └── init.sls
    ├── php
    │   └── init.sls
    └── top.sls

.. note::

    Briefly point out the differences in these two methods and the pros and
    cons. The latter is probably preferred if it will handle anything more than
    the single .sls file.


Example: /srv/salt/top.sls
--------------------------

.. code-block:: yaml

    base:
      'www*':
        - apache
        - mysql
        - php

This 'top file' instructs Salt to apply the 'apache', 'mysql', and 'php' states
to any machine matching the name 'www*'.

Example: apache/init.sls
------------------------

.. code-block:: yaml

    apache:
      pkg:
        - installed
        - name: httpd
      service:
        - running
        - name: httpd
        - enable: True
        - watch:
          - pkg: apache

This state file instructs Salt to install the apache package and to enable and
start the service. Note the 'name' declaration to specify the proper package and
service name.

.. note::

    To spread out the time a little bit, type these up on the virtual machine
    as you go over them.

Example: mysql/init.sls
-----------------------

.. code-block:: yaml

    mysqld:
      pkg:
        - installed
        - name: mysql-server
      service:
        - running
        - name: mysqld
        - enable: True
        - watch:
          - pkg: mysqld

    mysql:
      pkg.installed

This state file instructs Salt to install the mysql server package and to
enable and start the service. Again, notice the use of the name declaration to
define the package and service name for this distribution.

.. note::

    To spread out the time a little bit, type these up on the virtual machine
    as you go over them.

Example: php/init.sls
---------------------

.. code-block:: yaml

    php:
      pkg.installed

    php-mysql:
      pkg.installed

This state file instructs Salt to install the php and php-mysql packages.

.. note::

    To spread out the time a little bit, type these up on the virtual machine
    as you go over them.

    Do a state.highstate test=True after this slide

Content
-------

What about the site content? How do we configure that?

.. note::

    Start a short discussion about different ways to push down the site content.
    This could be file.recurse, git.latest, in-house package installation, etc.

file.recurse:
-------------

.. code-block:: yaml

    /srv/http/domain1.tld/html:
      file.recurse:
        source: salt://apache/html/

This example uses the ``file.recurse`` method to recursively copy all files in
a directory into a destination folder on the minion.

.. note::

    In this example we're going to deploy a Wordpress site. We'll check out the
    source via svn and then file.recurse the contents into place.

git.latest:
-----------

.. code-block:: yaml

    https://github.com/username/repo.git:
      git.latest:
        - target: /srv/http/domain1.tld/html

This example uses the ``git.latest`` method to checkout a git repository of the
site contents into a destination folder. This also supports branch, tag, or
revision ID.

.. note::

    In this example we're going to deploy the full site contents using git.
    We'll check out the latest version of the repository and place it where it
    needs to be.

====
Demo
====

=================
Practice : States
=================

Practice writing states by creating an ``apache``, ``mysql`` and ``php`` sls
files. Apply these to your web server in the ``top.sls``.

.. note::

    Give the students some time to type of some states from scratch. After some
    time mention how some students are on RHEL while others are on Debian. Mention
    compatibility of package names and services. The solution is Formulas.

========
Formulas
========

SaltStack Formulas allow you to use pre-built, cross-distro advanced states
instead of writing your own. Formulas are available from:

 - https://github.com/saltstack-formulas/

.. note::

    Take a look at some of the many formulas available on github. Don't go into
    much detail, but have a look at the list.

gitfs
-----

``gitfs`` allows you to use git repositories directly for your ``file_roots``. A
simple way to use SaltStack Formulas is through gitfs.

``gitfs`` also requires the ``GitPython`` or ``python-git`` package.

.. note::
    
    Make sure the students understand that using gitfs is completely optional.
    The alternative would be to checkout the repositories into the local
    file_roots.

/etc/salt/master
----------------

First, declare gitfs as a ``fileserver_backend`` in the master config. Note:
order here is important (bug?).

.. code-block:: yaml

    fileserver_backend:
      - roots
      - git

.. note::

    Mention that the example in the sample config file has them in reverse
    order. This is likely a bug, but until such time that it's fixed, the order on
    the slide is important.

/etc/salt/master (cont.)
------------------------

Next, list the git repositories you want to use.

.. code-block:: yaml

    gitfs_remotes:
      - git://github.com/saltstack-formulas/apache-formula.git
      - git://github.com/saltstack-formulas/mysql-formula.git
      - git://github.com/saltstack-formulas/php-formula.git

The Apache Formula
------------------

The Apache Formula currently supports the following:

 - apache
 - apache.mod_proxy
 - apache.mod_proxy_http
 - apache.mod_wsgi
 - apache.vhosts.standard
 - apache.debian_full

.. note::

    Take some time to look at each of the sls. Explain how ``apache.mod_proxy``
    translates into the .sls files. 

apache.vhosts.standard
----------------------

Let's take a minute to look at the ``apache.vhosts.standard`` part of the
Formula.

.. note::

    Open the github page and look at how the ``apache.vhosts.standard`` part of
    the formula works. Explain what it will generate, and the pillar data that can
    be used to customize it.

The MySQL Formula
-----------------

The MySQL Formula currently supports two states:

 - mysql.client
 - mysql.server

The PHP Formula
---------------

The PHP Formula currently supports the following:

 - php
 - php.apc
 - php.curl
 - php.fpm
 - php.gd
 - php.mcrypt
 - php.mysql
 - php.pear

Example /srv/salt/top.sls
-------------------------

.. code-block:: yaml

    base:
      'www*':
        - apache
        - apache.vhosts.standard
        - mysql.server
        - php
        - php.mysql

Example /srv/pillar/top.sls
---------------------------

.. code-block:: yaml

    base:
      'www*':
        - vhosts

Example /srv/pillar/vhosts.sls
------------------------------

.. code-block:: yaml

    apache:
      sites:
        domain1.tld:
          template_file: salt://apache/vhosts/standard.tmpl
          template_engine: jinja
        domain2.tld:
          template_file: salt://apache/vhosts/standard.tmpl
          template_engine: jinja
        domain3.tld:
          template_file: salt://apache/vhosts/standard.tmpl
          template_engine: jinja
        domain4.tld:
          template_file: salt://apache/vhosts/standard.tmpl
          template_engine: jinja
        ...

====
Demo
====
    
===================
Practice : Formulas
===================

Practice using SaltStack Formulas via gitfs by setting the ``apache``,
``mysql`` and ``php`` formulas as ``gitfs_remotes``. Apply theseto your web
server in the ``top.sls``.

================
Christer Edwards
================

 - http://intothesaltmine.org
 - #salt (cedwards)
 - cedwards@xmission.com
