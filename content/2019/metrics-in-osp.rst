Metrics provided by collectd in OSP
###################################
:date: 2019-05-20 13:40
:author: mrunge
:category: OpenStack
:tags: collectd, OpenStack
:slug: default-collect-metrics
:Status: draft

This article is intended to give an overview on metrics available in OSP via
collectd.

In a default configuration, collectd is provided via `kolla docker container`_.


+-----------------*
| Plugin | metric |
+=================+
| apache
+-
| bind
+-
| ceph
+-
| chrony
+-
| curl 



.. _`kolla docker container`: https://github.com/openstack/kolla/blob/master/docker/collectd/Dockerfile.j2
