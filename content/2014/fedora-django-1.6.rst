How to use Django-1.5 via parallel installable python-django15
##############################################################
:date: 2014-03-26 13:00
:author: mrunge
:category: Fedora
:tags: Fedora
:slug: fedora-django-1-6

Last week, the package python-django15 was approved. It's a Django package
to be installed in parallel to e.g python-django. Once you installed it, 
you can not use it directly. 

For example for python-django-horizon aka. OpenStack Dashboard 
I'd change the django.wsgi from `github`_
to
 
.. code-block:: python

 import logging
 import os
 import sys

 ####
 # explicitly require Django 1.5 or later
 from pkg_resources import require
 require('Django>=1.5,<1.6')
 ####

 import django.core.handlers.wsgi
 from django.conf import settings

 # Add this file path to sys.path in order to import settings
 sys.path.insert(0, os.path.join(os.path.dirname(os.path.realpath(__file__)), '../..'))
 os.environ['DJANGO_SETTINGS_MODULE'] = 'openstack_dashboard.settings'
 sys.stdout = sys.stderr

 DEBUG = False

 application = django.core.handlers.wsgi.WSGIHandler()

Note the two lines enclosed in the lines of hash signs. That's all you need
to use Django-1.5 from python-django15 and have Django-1.6 installed in parallel.


.. _github: https://github.com/openstack/horizon/blob/master/openstack_dashboard/wsgi/django.wsgi
