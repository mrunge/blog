Config snippets for collectd configured by TripleO
##################################################
:date: 2020-07-15 15:00
:author: mrunge
:category: OpenStack
:tags: OpenStack, collectd
:slug: collectd-config
:Status: published

This is mostly a brain dump for myself for later reference, but may
be also useful for others.

As I wrote in an earlier post_, collectd is configured on OpenStack
TripleO driven deployments by a config file.

.. code:: yaml

   parameter_defaults:
       CollectdExtraPlugins:
           - write_http
       ExtraConfig:
           collectd::plugin::write_http::nodes:
               collectd:
                   url: collectd1.tld.org
                   metrics: true
                   header: foobar

The collectd exec plugin comes handy when launching some third party
script. However, the config may be a bit tricky, for example to execute
`/usr/bin/true` one would insert

.. code:: yaml

   parameter_defaults:
       CollectdExtraPlugins:
        - exec
       ExtraConfig:
         collectd::plugin::exec::commands:
           healthcheck:
             user: "collectd"
             group: "collectd"
             exec: ["/usr/bin/true",]


.. _post: http://www.matthias-runge.de/2018/06/08/tripleo-collectd/

