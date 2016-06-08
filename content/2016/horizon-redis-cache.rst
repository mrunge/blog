Caching in Horizon with Redis
#############################
:date: 2016-01-19 11:50
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: horizon-redis-caching
:Status: published

Redis_ is a in-memory data structure store, which can be used as
cache and session backend.

I thought to give it a try for Horizon.
Installation is quite simple, either ``pip install django-redis``
or ``dnf --enablerepo=rawhide install python-django-redis``.

Then change ``openstack_dashboard/local/local_settings.py`` aka.
``/etc/openstack-dashboard/local_settings`` to incorporate

.. code:: python

    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
            }
        }
    }


Make sure you also have something like

.. code:: python

    SESSION_ENGINE = "django.contrib.sessions.backends.cache"

Finally, start your redis and restart httpd:

.. code:: bash

    systemctl start redis
    systemctl restart httpd

Voilà, now your Horizon should use Redis for caching.

.. _redis: http://redis.io/
