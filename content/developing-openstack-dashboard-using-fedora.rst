Developing OpenStack Dashboard using Fedora
###########################################
:date: 2012-11-20 14:02
:author: mrunge
:category: Fedora, Linux, OpenStack
:slug: developing-openstack-dashboard-using-fedora

Lately, I'm getting involved into developing OpenStack, especially it's
dashboard.

Many developers recommend using `devstack`_ to work on OpenStack. For
some reasons, I can't recommend that for Dashboard. My colleague `Dan
Berrange`_ has an `article with all dirty details`_.

Good news is, at least for OpenStack's dashboard, devstack is not
required in any case, it just runs on plain Fedora. I simply followed
the instructions provided for `Fedora's test day`_, and they worked
great for me, no rocket science and no engineering master required.

| Additionally I installed git and git-review (as root):
|  ``yum -y install git git-review``
|  then go to the location where you want to put the code, for me that's
|  ``cd ~/work``

| and execute
|  ``git clone https://github.com/openstack/horizon.git && cd horizon``

| This will get the source code from github, will create a new directory
named horizon, and change to that directory.

|  ``./manage.py runserver``

starts Django's built in development server on your host, per default on
port 8000.

To visit your shiny Dashboard, direct your webbrowser to
http://localhost:8000 .

When finished with your desired changes, you should submit them for
review. The process around that is covered here:
http://wiki.openstack.org/GerritWorkflow

.. _devstack: http://devstack.org/
.. _Dan Berrange: http://berrange.com/
.. _article with all dirty details: http://berrange.com/posts/2012/11/20/what-devstack-does-to-your-host-when-setting-up-openstack-on-fedora-17/
.. _Fedora's test day: https://fedoraproject.org/wiki/Test_Day:2012-09-18_OpenStack
