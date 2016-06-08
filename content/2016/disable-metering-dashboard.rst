Disable "Resource Usage"-dashboard in Horizon
#############################################
:date: 2016-01-22 9:05
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: disable-metering-dashboard
:Status: published

When using Horizon as Admin user, you probably saw the metering
dashboard, also known as "Resource Usage".

It internally uses Ceilometer; Ceilometer continuously collects
data from configured data sources. In a cloud environment, this
can quickly grow enormously. When someone visits the metering
dashboard in Horizon, Ceilometer then will accumulate requested data
on the fly.

On the medium term, Horizon should switch to use gnocchi_; in between,
if you're tired on waiting, just disable metering dasboard e.g by placing
the file like ``_99_disable_metering_dashboard.py`` to
``/usr/share/openstack-dashboard/openstack_dashboard/local/enabled``

.. code:: python

    # The slug of the panel to be added to HORIZON_CONFIG. Required.
    PANEL = 'metering'
    # The slug of the dashboard the PANEL associated with. Required.
    PANEL_DASHBOARD = 'admin'
    # The slug of the panel group the PANEL is associated with.
    PANEL_GROUP = 'admin'
    REMOVE_PANEL = True


Finally, restart httpd:

.. code:: bash

    systemctl restart httpd

.. _gnocchi: http://gnocchi.xyz/
