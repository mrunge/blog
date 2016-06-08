Pelican version 3.6
###################
:date: 2015-06-22 20:00
:author: mrunge
:category: Fedora
:tags: Fedora
:slug: pelican-3.6
:Status: published

Today, I updated pelican_ to version 3.6 and also added a python 3 subpackage.
You can test it in Fedora 22 by using

.. code-block:: bash

  dnf --enablerepo=updates-testing install python3-pelican

If you're already using pelican,

.. code-block:: bash

  dnf --enablerepo=updates-testing update python-pelican

Please leave feedback_

Upgrading from previous release might require a change:

#. Delete the `cache` folder, if exists

#. Add the following to your settings file:

.. code-block:: bash

  CACHE_CONTENT = True
  LOAD_CONTENT_CACHE = True

The full change list is available on `pelicans news`_ page.

.. _pelican: http://blog.getpelican.com
.. _feedback: https://admin.fedoraproject.org/updates/python-pelican-3.6.0-1.fc22
.. _`pelicans news`: http://blog.getpelican.com/category/news.html
