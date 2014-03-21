A stable and custom theme for Horizon
#####################################
:date: 2014-03-18 09:00
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: custom-theme-for-horizon

During the OpenStack Icehouse development cycle, we gained in Horizon the
ability to use additional python packages by simply dropping in a file
in the ``openstack_dashboard/enabled`` dir. 

In the rest of the article, I'll propose a method to customize the look and
feel, without breaking during package updates. It will work, if
you have a Horizon package in Icehouse release RC1 or later.

Fedora and `RDO`_ openstack-dashboard packages are installing the Dashboard into
``/usr/share/openstack-dashboard``.

First of all, we'll add a config ``openstack_dashboard/enabled/_99_theme.py``

.. code-block:: python

    # The name of the dashboard to be added to HORIZON['dashboards']. Required.
    DASHBOARD = 'theme'
    # If set to True, this dashboard will be set as the default dashboard.
    DEFAULT = False
    # A dictionary of exception classes to be added to HORIZON['exceptions'].
    ADD_EXCEPTIONS = {}
    # A list of applications to be added to INSTALLED_APPS.
    ADD_INSTALLED_APPS = ['openstack_dashboard.dashboards.theme']

There are two notable settings: ``DASHBOARD = 'theme'`` - this is the name
of the added dashboard. We'll come back to this later. The other one is
``ADD_INSTALLED_APPS = ['openstack_dashboard.dashboards.theme']``.
This is a python module, which can reside everywhere in the file system
(as long as it's in the python search path). I chose to put it directly
below ``openstack_dashboard/dashboards`` in a dir named ``theme``

Actually, I put a very minimum set of files into the ``theme`` directory:

::

    theme/
    |-- dashboard.py
    |-- __init__.py
    |-- models.py
    |-- static
    |   `-- dashboard
    |       |-- css
    |       |   `-- font-awesome.min.css
    |       |-- fonts
    |       |   |-- FontAwesome.otf
    |       |   |-- OpenSans-Regular-webfont.eot
    |       |   |-- OpenSans-Regular-webfont.svg
    |       |   |-- OpenSans-Regular-webfont.ttf
    |       |   `-- OpenSans-Regular-webfont.woff
    |       |-- img
    |       |   |-- bg-login.jpg
    |       |   |-- brand.svg
    |       |   `-- logo.svg
    |       `-- less
    |           |-- rcue
    |           |   |-- fonts.less
    |           |   |-- icons.less
    |           |   |-- login.less
    |           |   |-- navbar.less
    |           `-- theme.less
    |-- templates
    |   |-- auth
    |   |   |-- _login.html
    |   |   `-- login.html
    |   |-- base.html
    |   |-- _header.html
    |   |-- horizon
    |   |   |-- common
    |   |   |   `-- _sidebar.html
    |   |   |-- _nav_list.html
    |   |   `-- _subnav_list.html
    |   |-- splash.html
    |   `-- _stylesheets.html
    `-- theme_index
        |-- __init__.py
        |-- __init__.pyc
        |-- panel.py
        |-- panel.pyc
        |-- urls.py
        |-- urls.pyc
        |-- views.py
        `-- views.pyc

At first glance, that looks way more than it actually is. While most
files are self-explanatory, I'll go into details with a few files.

theme-directory
---------------

The files ``__init__.py`` and ``models.py`` may be completely empty, or 
just contain a comment:

.. code-block:: python

    # intentionally left blank

The more interesting file is dashboard.py:

.. code-block:: python

    import horizon
    
    
    class Theme(horizon.Dashboard):
        name = _("theme")
        slug = "theme"
        panels = ('theme_index', )
        default_panel = 'theme_index'
        nav = False

    horizon.register(Theme)

Important to note is the option ``nav = False``, which prevents this dashboard
to show up in the navigation bar; this is used e.g for contents to be linked
manually, like "Settings" or not to be linked at all, like a theme.

theme_index
-----------

As above in the ``theme`` dir, ``__init__.py``  and ``views.py`` may be
completely empty. ``urls.py`` is nearly empty as well:

.. code-block:: python

    urlpatterns = ()

``panels.py``:

.. code-block:: python

    import horizon

    from openstack_dashboard.dashboards.theme import dashboard

    class ThemePanel(horizon.Panel):
        name = "Panel providing a theme"
        slug = 'theme_index'
        nav = True

    dashboard.Theme.register(ThemePanel)

This code snippet should be enough for Horizon to think, there is a real
dashboard, which just should not be included in the automatically generated
navigation.

Putting all together
--------------------

Now we can put templates into the ``templates`` directory, and static files i
to ``static``. When Horizon delivers web pages, based on templates, it will 
search for the them in ``theme/templates`` first; if found there, they will
be delivered, if not, Horizon will fall back to the default pages found in the
other ``templates`` dirs in horizon source tree.

As a starter, just copy e.g ``base.html`` or ``_stylesheets.html`` from 
``horizon/templates`` directory, to ``templates`` and modify them.

When adding static files, you need to issue

| ``./manage.py collectstatic``

from ``/usr/share/openstack-dashboard`` to collect them and to copy
static files to ``/usr/share/openstack-dashboard/static``.

.. _RDO: http://openstack.redhat.com/Main_Page
