Title: Debugging OpenStack Metrics
Date: 2021-12-06 16:00
Author: mrunge
Category: OpenStack
Tags: OpenStack, Telemetry
Slug: debugging-openstack-metrics
Status: published

## Overview


We'll do the following steps:

1. [check if metrics are getting in](#metric_get_in)
2. [if not, check if ceilometer is running](#ceilometer_running)
3. [check if gnocchi is running](#gnocchi_running)
4. [query gnocchi for metrics](#query_gnocchi)
5. [check alarming](#query_alarms)
6. [further debugging](#further_debugging)



### Check that metrics are getting in {#metric_get_in}

`openstack server list`

example:

```
$ openstack server list  -c ID -c Name -c Status
+--------------------------------------+-------+--------+
| ID                                   | Name  | Status |
+--------------------------------------+-------+--------+
| 7e087c31-7e9c-47f7-a4c4-ebcc20034faa | foo-4 | ACTIVE |
| 977fd250-75d4-4da3-a37c-bc7649047151 | foo-2 | ACTIVE |
| a1182f44-7163-4ad9-89ed-36611f75bac7 | foo-5 | ACTIVE |
| a91951b3-ff4e-46a0-a5fd-20064f02afc9 | foo-3 | ACTIVE |
| f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f | foo-1 | ACTIVE |
+--------------------------------------+-------+--------+
```

We'll use `f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f`. That's the uuid of the server `foo-1`. For convenience, the server uuid and the resource ID used in `openstack metric resource` are the same.

```
$ openstack metric resource show f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f
+-----------------------+-------------------------------------------------------------------+
| Field                 | Value                                                             |
+-----------------------+-------------------------------------------------------------------+
| created_by_project_id | 6a180caa64894f04b1d04aeed1cb920e                                  |
| created_by_user_id    | 39d9e30374a74fe8b58dee9e1dcd7382                                  |
| creator               | 39d9e30374a74fe8b58dee9e1dcd7382:6a180caa64894f04b1d04aeed1cb920e |
| ended_at              | None                                                              |
| id                    | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f                              |
| metrics               | cpu: fe84c113-f98d-42ee-84fd-bdf360842bdb                         |
|                       | disk.ephemeral.size: ad79f268-5f56-4ff8-8ece-d1f170621217         |
|                       | disk.root.size: 6e021f8c-ead0-46e4-bd26-59131318e6a2              |
|                       | memory.usage: b768ec46-5e49-4d9a-b00d-004f610c152d                |
|                       | memory: 1a4e720a-2151-4265-96cf-4daf633611b2                      |
|                       | vcpus: 68654bc0-8275-4690-9433-27fe4a3aef9e                       |
| original_resource_id  | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f                              |
| project_id            | 8d077dbea6034e5aa45c0146d1feac5f                                  |
| revision_end          | None                                                              |
| revision_start        | 2021-11-09T10:00:46.241527+00:00                                  |
| started_at            | 2021-11-09T09:29:12.842149+00:00                                  |
| type                  | instance                                                          |
| user_id               | 65bfeb6cc8ec4df4a3d53550f7e99a5a                                  |
+-----------------------+-------------------------------------------------------------------+
```

This list shows the metrics associated with the instance.

You are done here.

### Checking if ceilometer is running {#ceilometer_running}

```
$ ssh controller-0 -l root
$ podman ps --format "{{.Names}} {{.Status}}" | grep ceilometer
ceilometer_agent_central Up 2 hours ago
ceilometer_agent_notification Up 2 hours ago
```

On compute nodes, there should be ceilometer_agent_compute running

```
$ podman ps --format "{{.Names}} {{.Status}}" | grep ceilometer
ceilometer_agent_compute Up 2 hours ago
```

The metrics are being sent from ceilometer to a remote defined in 
`/var/lib/config-data/puppet-generated/ceilometer/etc/ceilometer/pipeline.yaml`
, which may look similar to the following file

```
---
sources:
    - name: meter_source
      meters:
          - "*"
      sinks:
          - meter_sink
sinks:
    - name: meter_sink
      publishers:
          - gnocchi://?filter_project=service&archive_policy=ceilometer-high-rate
          - notifier://172.17.1.40:5666/?driver=amqp&topic=metering

```
In this case, data is sent to both STF and Gnocchi. Next step is to check 
if there are any errors happening. On controllers and computes, ceilometer 
logs are found in `/var/log/containers/ceilometer/`. 

The `agent-notification.log` shows logs from publishing data, as well as
errors if sending out metrics or logs fails for some reason.

If there are any errors in the log file, it is likely that metrics are not
being delivered to the remote.

```
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging   File "/usr/lib/python3.6/site-packages/oslo_messaging/transport.py", line 136, in _send_notification
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging     retry=retry)
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging   File "/usr/lib/python3.6/site-packages/oslo_messaging/_drivers/impl_amqp1.py", line 295, in wrap
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging     return func(self, *args, **kws)
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging   File "/usr/lib/python3.6/site-packages/oslo_messaging/_drivers/impl_amqp1.py", line 397, in send_notification
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging     raise rc
2021-11-16 07:01:07.063 16 ERROR oslo_messaging.notify.messaging oslo_messaging.exceptions.MessageDeliveryFailure: Notify message sent to <Target topic=event.sample> failed: timed out
```

In this case, it failes to send messages to the STF instance. The following
example shows the gnocchi api not responding or not being accessible

```
2021-11-16 10:38:07.707 16 ERROR ceilometer.publisher.gnocchi [-] <html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
 (HTTP 503): gnocchiclient.exceptions.ClientException: <html><body><h1>503 Service Unavailable</h1>
```

For more gnocchi debugging, see the gnocchi section.

### Gnocchi {#gnocchi_running}

Gnocchi sits on controller nodes and consists of three separate containers,
gnocchi_metricd, gnocchi_statsd, and gnocchi_api. The latter is for the
interaction with the outside world, such as ingesting metrics or returning
measurements.

Gnocchi metricd are used for re-calculating metrics, downsampling for lower
granularity, etc. Gnocchi logfiles are found under `/var/log/containers/gnocchi`
and the gnocchi API is hooked into httpd, thus the logfiles are
stored under `/var/log/containers/httpd/gnocchi-api/`. The corresponding files
there are either `gnocchi_wsgi_access.log` or `gnocchi_wsgi_error.log`.

In the case from above (ceilometer section), where ceilometer could not
send metrics to gnocchi, one would also observe log output for the
gnocchi API.

### Retrieving metrics from Gnocchi {#query_gnocchi}

For a starter, let's see which resources there are.

```
openstack server list -c ID -c Name -c Status
+--------------------------------------+-------+--------+
| ID                                   | Name  | Status |
+--------------------------------------+-------+--------+
| 7e087c31-7e9c-47f7-a4c4-ebcc20034faa | foo-4 | ACTIVE |
| 977fd250-75d4-4da3-a37c-bc7649047151 | foo-2 | ACTIVE |
| a1182f44-7163-4ad9-89ed-36611f75bac7 | foo-5 | ACTIVE |
| a91951b3-ff4e-46a0-a5fd-20064f02afc9 | foo-3 | ACTIVE |
| f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f | foo-1 | ACTIVE |
+--------------------------------------+-------+--------+
```

To show which metrics are stored for the vm `foo-1` one would use the following
command

```
openstack metric resource show f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f --max-width 75
+-----------------------+-------------------------------------------------+
| Field                 | Value                                           |
+-----------------------+-------------------------------------------------+
| created_by_project_id | 6a180caa64894f04b1d04aeed1cb920e                |
| created_by_user_id    | 39d9e30374a74fe8b58dee9e1dcd7382                |
| creator               | 39d9e30374a74fe8b58dee9e1dcd7382:6a180caa64894f |
|                       | 04b1d04aeed1cb920e                              |
| ended_at              | None                                            |
| id                    | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f            |
| metrics               | cpu: fe84c113-f98d-42ee-84fd-bdf360842bdb       |
|                       | disk.ephemeral.size:                            |
|                       | ad79f268-5f56-4ff8-8ece-d1f170621217            |
|                       | disk.root.size:                                 |
|                       | 6e021f8c-ead0-46e4-bd26-59131318e6a2            |
|                       | memory.usage:                                   |
|                       | b768ec46-5e49-4d9a-b00d-004f610c152d            |
|                       | memory: 1a4e720a-2151-4265-96cf-4daf633611b2    |
|                       | vcpus: 68654bc0-8275-4690-9433-27fe4a3aef9e     |
| original_resource_id  | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f            |
| project_id            | 8d077dbea6034e5aa45c0146d1feac5f                |
| revision_end          | None                                            |
| revision_start        | 2021-11-09T10:00:46.241527+00:00                |
| started_at            | 2021-11-09T09:29:12.842149+00:00                |
| type                  | instance                                        |
| user_id               | 65bfeb6cc8ec4df4a3d53550f7e99a5a                |
+-----------------------+-------------------------------------------------+
```

To view the memory usage between Nov 18 2021 17:00 UTC and 17:05 UTC, one
would issue this command:

```
openstack metric measures show --start 2021-11-18T17:00:00 \
                               --stop 2021-11-18T17:05:00 \
                               --aggregation mean 
                               b768ec46-5e49-4d9a-b00d-004f610c152d

+---------------------------+-------------+-------------+
| timestamp                 | granularity |       value |
+---------------------------+-------------+-------------+
| 2021-11-18T17:00:00+00:00 |      3600.0 | 28.87890625 |
| 2021-11-18T17:00:00+00:00 |        60.0 | 28.87890625 |
| 2021-11-18T17:01:00+00:00 |        60.0 | 28.87890625 |
| 2021-11-18T17:02:00+00:00 |        60.0 | 28.87890625 |
| 2021-11-18T17:03:00+00:00 |        60.0 | 28.87890625 |
| 2021-11-18T17:04:00+00:00 |        60.0 | 28.87890625 |
| 2021-11-18T17:00:14+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:00:44+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:01:14+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:01:44+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:02:14+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:02:44+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:03:14+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:03:44+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:04:14+00:00 |         1.0 | 28.87890625 |
| 2021-11-18T17:04:44+00:00 |         1.0 | 28.87890625 |
+---------------------------+-------------+-------------+
```

This shows, the data is available with granularity 3600, 60 and 1 sec. The memory usage does
not change over the time, that's why the values don't change. Please
note, if you'd be asking for values with the granularity of 300,
    the result will be empty
```
$ openstack metric measures show --start 2021-11-18T17:00:00 \
              --stop 2021-11-18T17:05:00 \
              --aggregation mean \
              --granularity 300
              b768ec46-5e49-4d9a-b00d-004f610c152d
Aggregation method 'mean' at granularity '300.0' for metric b768ec46-5e49-4d9a-b00d-004f610c152d does not exist (HTTP 404)
```

More info about the metric can be actually listed by using

```
openstack metric show --resource-id f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f \
              memory.usage \
              --max-width 75

+--------------------------------+----------------------------------------+
| Field                          | Value                                  |
+--------------------------------+----------------------------------------+
| archive_policy/name            | ceilometer-high-rate                   |
| creator                        | 39d9e30374a74fe8b58dee9e1dcd7382:6a180 |
|                                | caa64894f04b1d04aeed1cb920e            |
| id                             | b768ec46-5e49-4d9a-b00d-004f610c152d   |
| name                           | memory.usage                           |
| resource/created_by_project_id | 6a180caa64894f04b1d04aeed1cb920e       |
| resource/created_by_user_id    | 39d9e30374a74fe8b58dee9e1dcd7382       |
| resource/creator               | 39d9e30374a74fe8b58dee9e1dcd7382:6a180 |
|                                | caa64894f04b1d04aeed1cb920e            |
| resource/ended_at              | None                                   |
| resource/id                    | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f   |
| resource/original_resource_id  | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f   |
| resource/project_id            | 8d077dbea6034e5aa45c0146d1feac5f       |
| resource/revision_end          | None                                   |
| resource/revision_start        | 2021-11-09T10:00:46.241527+00:00       |
| resource/started_at            | 2021-11-09T09:29:12.842149+00:00       |
| resource/type                  | instance                               |
| resource/user_id               | 65bfeb6cc8ec4df4a3d53550f7e99a5a       |
| unit                           | MB                                     |
+--------------------------------+----------------------------------------+
```

It shows in this case, the used archive policy is ceilometer-high-rate.

```
openstack metric archive-policy show ceilometer-high-rate --max-width 75
+---------------------+---------------------------------------------------+
| Field               | Value                                             |
+---------------------+---------------------------------------------------+
| aggregation_methods | mean, rate:mean                                   |
| back_window         | 0                                                 |
| definition          | - timespan: 1:00:00, granularity: 0:00:01,        |
|                     | points: 3600                                      |
|                     | - timespan: 1 day, 0:00:00, granularity: 0:01:00, |
|                     | points: 1440                                      |
|                     | - timespan: 365 days, 0:00:00, granularity:       |
|                     | 1:00:00, points: 8760                             |
| name                | ceilometer-high-rate                              |
+---------------------+---------------------------------------------------+
```

That means, in this case, the aggregation methods one could use for querying
the metrics are just mean and rate:mean. Other methods could include min or max.

### Alarming {#query_alarms}

Alarms can be retrieved by issuing

```
$ openstack alarm list


```

To create an alarm, for example based on disk.ephemeral.size, one would use something
like

```
openstack alarm create --alarm-action 'log://' \
              --ok-action 'log://' \
              --comparison-operator ge \
              --evaluation-periods 1 \
              --granularity 60 \
              --aggregation-method mean \
              --metric disk.ephemeral.size \
              --resource-id f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f \
              --name ephemeral \
              -t gnocchi_resources_threshold \
              --resource-type instance \
              --threshold 1

+---------------------------+----------------------------------------+
| Field                     | Value                                  |
+---------------------------+----------------------------------------+
| aggregation_method        | mean                                   |
| alarm_actions             | ['log:']                               |
| alarm_id                  | 994a1710-98e8-495f-89b5-f14349575c96   |
| comparison_operator       | ge                                     |
| description               | gnocchi_resources_threshold alarm rule |
| enabled                   | True                                   |
| evaluation_periods        | 1                                      |
| granularity               | 60                                     |
| insufficient_data_actions | []                                     |
| metric                    | disk.ephemeral.size                    |
| name                      | ephemeral                              |
| ok_actions                | ['log:']                               |
| project_id                | 8d077dbea6034e5aa45c0146d1feac5f       |
| repeat_actions            | False                                  |
| resource_id               | f0a62c20-2304-4c8c-aaf5-f5bc9d385b5f   |
| resource_type             | instance                               |
| severity                  | low                                    |
| state                     | insufficient data                      |
| state_reason              | Not evaluated yet                      |
| state_timestamp           | 2021-11-22T10:16:15.250720             |
| threshold                 | 1.0                                    |
| time_constraints          | []                                     |
| timestamp                 | 2021-11-22T10:16:15.250720             |
| type                      | gnocchi_resources_threshold            |
| user_id                   | 65bfeb6cc8ec4df4a3d53550f7e99a5a       |
+---------------------------+----------------------------------------+
```

The state here `insufficient data` states, the data gathered or
stored is not sufficient to compare against. There is also a state
reason given, in this case `Not evaluated yet`, which gives an explanation.

Another valid reason could be `No datapoint for granularity 60`. 


### Further debugging {#further_debugging}

On OpenStack installations deployed via Tripleo aka OSP Director, the log files are located
on the separate nodes under `/var/log/containers/{service_name}/`. The config files for
the services are stored under `/var/lib/config-data/puppet-generated/<service_name>`
and are mounted into the containers.
