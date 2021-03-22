Author: mrunge
Title: High memory usage with collectd
Category: collectd
Tags: OpenStack, collectd
Slug: collectd-memory-usage
Status: published
Date: 2021-03-22 16:00

collectd itself is intended as lightweight collecting agent for metrics
and events. In larger infrastructure, the data is sent over the network
to a central point, where data is stored and processed further.

This introduces a potential issue: what happens, if the remote endpoint
to write data to is not available. The traditional network plugin uses
UDP, which is by definition unreliable.

Collectd has a queue of values to be written to an output plugin, such
was `write_http` or `amqp1`. At the time, when metrics should be
written, collectd iterates on that queue and tries to write this data
to the endpoint. If writing was successful, the data is removed from
the queue. The little word *if* also hints, there is a chance that data
doesn't get removed. The question is: what happens, or what should be
done?

There is no easy answer to this. Some people tend to ignore missed
metrics, some don't. The way to address this is to cap the queue at a
given length and to remove oldest data when new comes in. The parameters
are `WriteQueueLimitHigh` and `WriteQueueLimitLow`. If they are unset,
the queue is not limited and will grow until memory is out. For
predictability reasons, you should set these two values to the same
number. To get the *right* value for this parameter, it would require a
bit of experimentation. If values are dropped, one would see that in
the log file.

When collectd is configured as part of Red Hat OpenStack Platform, the
following config snippet can be used:

```
parameter_defaults:
    ExtraConfig:
      collectd::write_queue_limit_high: 100
      collectd::write_queue_limit_low: 100
```

Another parameter can be used to limit explicitly the queue length in
case the amqp1 plugin is used for sending out data: the `SendQueueLimit`
parameter, which is used for the same purpose, but can differ from the
global `WriteQueueLimitHigh` and `WriteQueueLimitLow`.

```
parameter_defaults:
    ExtraConfig:
        collectd::plugin::amqp1::send_queue_limit: 50
```

In almost all cases, the issue of collectd using much memory could be
tracked down to a write endpoint not being available, dropping data
occasionally, etc.
