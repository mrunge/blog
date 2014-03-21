How to create a custom theme for Horizon
########################################
:date: 2013-02-15 17:09
:author: mrunge
:category: Fedora, Linux, OpenStack
:tags: OpenStack
:slug: how-to-create-a-custom-theme-for-horizon

Based on the idea or proposal of Kieran Spear, I'll document here, what
has to be done, if one needs to customize Horizon (for whatever reason).

Let's say, one needs to create a corporate theme, named
horizon\_bigcorp. Then we first need to change local\_settings.py. In
Fedora and EPEL, that file is located at
``/etc/openstack-dashboard/local_settings``

Append the following lines to that file

::

    THEME_APP = 'horizon_bigcorp'

    try:
        __import__(THEME_APP)
        INSTALLED_APPS = (THEME_APP,) + INSTALLED_APPS
    except:
        pass

| Then we'll create a customization app:

::

    ├── __init__.py
    ├── __init__.pyc
    ├── models.py
    ├── models.pyc
    ├── static
    │   └── dashboard
    │       ├── img
    │       │   ├── bigcorpfavicon.ico
    │       │   └── bigcorplogo.png
    │       └── less
    │           └── bigcorptheme.less
    └── templates
        ├── horizon
        │   └── common
        │       └── _sidebar.html
        └── _stylesheets.html

    7 directories, 9 files

| Those .py files just need to be empty files.
|  The magic is in ``_stylesheets.html``, it's basically a modified copy
of file ``openstack_dashboard/templates/_stylesheets.html``:

::

    {% load compress %}

    {% compress css %}
    <link href='{{ STATIC_URL }}dashboard/less/horizon.less' type='text/less' media='screen' rel='stylesheet' />
    {% endcompress %}

    <link rel="shortcut icon" href="{{ STATIC_URL }}dashboard/img/favicon.ico"/>

Note, you just need a customized theme.less, if you did changes at css
(most likely). Like stylesheets, theme.less is a modified copy of it's
counterpart taken from the openstack\_dashboard directory.

Another example, how this can be implemented can be found here:
https://github.com/mrunge/horizon/tree/customization, where I modified
``settings.py`` instead of changing local\_settings.py.example
