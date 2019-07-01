Configuring collectd plugins with TripleO
#########################################
:date: 2018-06-08 08:40
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: tripleo-collectd
:Status: published

A way of deploying OpenStack is to use TripleO. This takes the an
approach to deploy a small OpenStack environment, and then to take
OpenStack provided infrastructure and tools to deploy the actual
production environment. This is actually done by an addition to the
openstack command line client:

.. code:: bash

    openstack overcloud deploy \
        --templates /usr/share/openstack-tripleo-heat-templates \
        -e ....


For performance metrics, there are a few `collectd plugins`_ configured
by default. If there is a need for more, just add them to
``CollectdExtraPlugins``, as shown below. Usually, this would be done by
creating a file, e.g ``collectd.yaml`` and make it read like the following
example.

.. code:: yaml

    parameter_defaults:
        CollectdExtraPlugins:
            - cpu
            - disk
            - df
            - hugepages
            - interface
            - memory
            - ovs_events
            - ovs_stats
            - processes
            - virt

    ExtraConfig:
        collectd::plugin::df::ignoreselected: true
        collectd::plugin::interface::ignoreselected: true
        collectd::plugin::virt::connection: "qemu:///system"
        collectd::plugin::virt::hostname_format: "hostname uuid"

Finally, if you saved ``collectd.yaml`` to ``/home/stack/collectd.yaml``
one would call it, like in the following code

.. code:: bash


    openstack overcloud deploy \
        --templates /usr/share/openstack-tripleo-heat-templates \
        ...
        -e /usr/share/openstack-tripleo-heat-templates/environments/services-docker/collectd.yaml \
        -e /home/stack/collectd.yaml

.. _`collectd plugins`: https://collectd.org/wiki/index.php/Table_of_Plugins
