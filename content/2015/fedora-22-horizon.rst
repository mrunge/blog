Horizon in Kilo version coming to Fedora 22
###########################################
:date: 2015-04-17 10:30
:author: mrunge
:category: OpenStack
:tags: Fedora, OpenStack
:slug: horizon-kilo-fedora22
:Status: draft

Testing horizon git checkouts
-----------------------------

One of the cool things in Horizon is, one can easily try newest things out.
This assumes, you already have an OpenStack installation available somewhere.

If you haven't already installed git:

.. code-block:: bash

  yum install git

To clone horizons upstream git repository, run the command

.. code-block:: bash

  git clone https://github.com/openstack/horizon
  cd horizon
  cp openstack_dashboard/local/local_settings.py.example openstack_dashboard/local/local_settings.py


Then please edit the file ``openstack_dashboard/local/local_settings.py``
and adjust 

.. code-block:: python

  ALLOWED_HOSTS = ['*']
  
  ...
  
  OPENSTACK_HOST = "127.0.0.1"


``OPENSTACK_HOST`` has to point to your keystone instance, in this case to 
127.0.0.1.

The development server requires a dependencies. If you didn't already
install them, just run:

.. code-block:: bash

  yum install gcc python-devel python-virtualenv openssl-devel libffi-devel which

To start your development server:

.. code-block:: bash

  ./run_tests.sh -m collectstatic

This will install all required dependencies in a virtual env inside your
horizon directory. Horizon spreads static files like images and 
javascript files all around in its source tree. ``collectstatic`` will
collect them and place those files in under a directory named static
inside the horizon checkout. When running the development server, they
will be served from that location.

Finally

.. code-block:: bash

  ./run_tests.sh --runserver

will start your Horizon instance from your git checkout. It can be accessed
via ``http://<ip>:8000``, in most cases, that is http://localhost:8000 .



Updating your checkout
----------------------

The following snippet will pull in latest changes from git, will copy
changed static files to the right places and will run your django development
server.
.. code-block:: bash

  git fetch && git pull
  ./run_tests.sh -m collectstatic
  ./run_tests.sh --runserver
