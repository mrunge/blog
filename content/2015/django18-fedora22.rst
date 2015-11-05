Fedora 22 will contain Django-1.8
#################################
:date: 2015-04-17 11:45
:author: mrunge
:category: Fedora
:tags: Fedora, OpenStack
:slug: django18-fedora22
:Status: published

One of the new features in upcoming Fedora 22 will be Django-1.8_. 
Django project released its most recent version earlier this month, and
it's going to be a long term supported version after Django-1.4 became a 
bit ancient nowadays. 
Fedora had release 1.6 in which is now deprecated by Django upstream.

A few packages required patches to support Django-1.8 though:

* python-django-openstack-auth

* python-django-compressor

* python-django-nose

* python-django-horizon

Of course, they were fixed and included already in Fedora 22.

.. _Django-1.8: https://fedoraproject.org/wiki/Changes/Django18
