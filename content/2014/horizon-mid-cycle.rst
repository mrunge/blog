Horizon's new features introduced in Juno cycle
###############################################
:date: 2014-09-03 13:55
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: horizon-juno-cycle-features
:status: draft

This post intends to give an overview on what happened during Horizons
Juno development cycle. Horizon's `blueprints page on launchpad`_ lists 31 
implemented new features. They may be grouped into a few larger features.

Sahara-Dashboard
----------------
Apache Hadoop is a widely adopted MapReduce implementation. The aim of the
Sahara project is to enable users to easily provision and manage Hadoop 
clusters on OpenStack.

During Juno development cycle, the independant Sahara dashboard was merged into
Horizon. Like all other features in Horizon, it will be shown, when Sahara is
configured through Keystone.

RBAC
----

Horizon-2014.1 aka Icehouse version has support for RBAC for Glance and Cinder.
For example, create, access, or delete images can be limited on user
or role basis. 
In Juno this RBAC system was extended to support `compute`_, `network`_, and 
`provisioning`_.

JavaScript unbundling
---------------------

In the past, there were quite a few JavaScript libraries copied into Horizon's 
code. Benefit is, they are available directly in Horizon and developers are
in control of them. On the other side, if there is a security flaw in any of
those bundled files, Horizon devs are in charge to fix it. Many Linux 
distributions have a rule not to use such bundled code at all, for 
example `Fedora`_. As result of this feature, all bundled libraries were
removed from Horizon, and system provided libraries will be used through
python-XStatic.

Horizon was originally intended as a framework to enable the development
of a dashboard, like the now well known OpenStack Dashboard, which is 
widely known under the name Horizon.

Other projects could use, Horizon's framework as well. There is a blueprint
described on launchpad, to `separate horizon from openstack dashboard`_. To 
enable this feature, JavaScript libraries need to become separate as well.


Look and feel improvements
--------------------------

There are quite a few blueprints striving to improve look and feel. For 
example, bootstrap was updated to version 3, the color palette was centralized.
The tablesorter plugin gained a feature to sort by timestamp. Those 
features make it easier to customize Horizon for individual needs.

Even more
---------

A few patches were added to enable to store metadata, like in cinder or glance,
where users can register key/pair values to describe their cloud deployment.
Another set of patches implements features for Neutron, like supporting IPv6 
and Neutron subnets.

.. _`blueprints page on launchpad`: https://blueprints.launchpad.net/horizon/juno
.. _`compute`: https://blueprints.launchpad.net/horizon/+spec/compute-rbac
.. _`network`: https://blueprints.launchpad.net/horizon/+spec/network-rbac
.. _`provisioning`: https://blueprints.launchpad.net/horizon/+spec/heat-rbac
.. _`Fedora`: https://fedoraproject.org/wiki/Packaging:No_Bundled_Libraries
.. _`separate horizon from openstack dashboard`: https://blueprints.launchpad.net/horizon/+spec/separate-horizon-from-dashboard
