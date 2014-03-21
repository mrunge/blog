Overview on Horizons new features introduced in Juno cycle
##########################################################
:date: 2014-08-29 09:00
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: horizon-mid-cycle-features
:status: draft

This post intends to give an overview on what happened during Horizons
Juno development cycle. The `blueprints page on launchpad`_ lists 31 implemented
new features. They may be grouped into a few larger features.

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
For example, create, access, or delete images is can be limited on user
or role basis. 
In Juno was this RBAC system extended to support `compute`_, `network`_, and 
`provisioning`_.

JavaScript unbundling
---------------------




.. _`blueprints page on launchpad`: https://blueprints.launchpad.net/horizon/juno
.. _`compute`: https://blueprints.launchpad.net/horizon/+spec/compute-rbac
.. _`network`: https://blueprints.launchpad.net/horizon/+spec/network-rbac
.. _`provisioning`: https://blueprints.launchpad.net/horizon/+spec/heat-rbac
