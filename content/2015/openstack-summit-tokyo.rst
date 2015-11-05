OpenStack Summit Tokyo
######################
:date: 2015-11-05 13:30
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: openstack-summit-tokyo
:Status: published

Over the past week, I attended the `OpenStack Summit`_ in Tokyo.

My primary
focus was on Horizon sessions. Nevertheless, I was lucky to
have one or two glimpses at more touristic spots in Tokyo.

.. _`OpenStack Summit`: https://www.openstack.org/summit/tokyo-2015/

.. image:: http://matthias-runge.de/fedora/temple.jpg
   :height: 300
   :width: 533
   :alt: temple in the area


Plugins
-------

The first work session for Horizon started on Wednesday with plugins.
Previously, there has been a session presenting a way to provide
plugins_. It looks nearly finished, all it really needs, is documentation.
The discussion mostly focused on issues for distributors; currently, policy
checks only work, if plugins were added by addition to ``local_settings.py``;
unfortunately, that is not an option for distributors. The discussion then
went to changing configuration to ``.ini`` files. The issue with it is,
configuration schema in horizon is pretty complex, and that won't change
with the `proposed approach`_. OpenStack distributors can't allow
automated changes to configuration files.

Most important piece for the rest of the session was, we agreed on moving
content out of Horizon core, such as

* Sahara
* Trove
* Heat
* Ceilometer

Metering support in Horizon has been quite unusable, as it takes very long
to search or aggregate collected data. The other contents were pretty much
blocked in the past, because they require significant installations and
configurations. Horizon cores were unable to provide constant reviews on
content esp. for Trove and Sahara.

.. _plugins: https://www.youtube.com/watch?v=Km99BCHfBdk
.. _`proposed approach`: https://review.openstack.org/#/c/100521/


Existential Crisis, AngularJS or not  to AngularJS
--------------------------------------------------

The AngularJS kind of stalled during Liberty development cycle. In total,
about 300 patches have landed; it has been the observation, the new
Angular pages are slower than traditional Django pages. One reason for this
seems to be, all JavasScript is loaded on each page load, no matter if
the code is really required or not. One answer to this was to move to a
single page application as soon as possible.
Another issue is, docs or a more complete example is still missing.
Functional tests are run on existing panels, Angular panels are ignored.

How to make meaningful progress with AngularJS
----------------------------------------------

This session could be seen as continuation of previous session. Final
outcome was, we'll focus on 2-3 panels in AngularJS; new panels should
be proposed with ``DISABLED = True`` setting. New launch instance
workflow is still not enabled by default, currently there are 5 patches
up for review fixing bugs. The priority should lie on fixing issues
and stabilizing existing panels before pushing more Angular-based panels
to Horizon.


Horizon and async features
--------------------------

Horizon does not use any message bus; for updates on asynchronous operations
a pulling mechanism is used. In this scope, an asynchronous operation is
any longer running operation, like launching an instance, create a volume
etc. The underlying API returns immediately without returning the actual
status of the operation, just something like "operation scheduled". One idea
to solve this, is to utilize zaqar as a message bus.

Setting Mitaka priorities and non-priorities
--------------------------------------------

Finally, we decided on priorities_ for Mitaka cycle. Most important areas to
work on will be

* Plugins, documentation is missing
* Angular content

  * Users panel

  * Images panel
* Theming, documentation is missing
* Dependencies, currently pbr is blocking to release XStatic packages
* Performance

The linked etherpad page will be used during Mitaka development cycle
to coordinate the work, like we used an etherpad over Liberty cycle.

.. _priorities: https://etherpad.openstack.org/p/mitaka-horizon-priorities
