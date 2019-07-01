Collectd contributors meetup in Brussels
########################################
:date: 2019-02-04 11:40
:author: mrunge
:category: OpenStack
:tags: collectd
:slug: collectd-meetup-bru
:Status: published

Last week, on January 31 and Feb 01, we had a contributors meetup of
collectd_ in Brussels, just before FOSDEM_. Thanks to our generous
sponsors Camptocamp_, Intel_, and `Red Hat`_, we were able to meet
and to talk about various topics. In total, there were 15 persons
attending, and if I'm not mistaken, all of them were sent by their
employers, which underlines the interest here again.

 * There was a request or suggestion, to have a monthly video call or
   a chat irc meeting to talk about progress, or to address issues
   raised over reviews.
   That could be a joint call monthly with the `OPNFV Barometer`_
   community. More info and also minutes are available_.

 * We also talked about the concept of `code-ownership`_, and decided
   to give it a try.

 * Although the wiki registration is disabled, we would continue with using
   the current mediawiki for now. Octo pointed out, that spammers were
   causing real issues and work. If there is a request for getting a wiki
   account, please get in touch with one of the maintainers.

 * In the session we had about testing and reviving CI tests, we briefly
   discussed both functional and also integration tests. Especially the
   latter will require some investigation. Marc already started to set-up
   and fix the CI for collectd on github.

 * In the release process session, we went through the current release
   process and talked about possible automations, or how to make it a less
   time consuming process. Especially writing release notes or the changelog
   took time in the past. At the end, we decided to target three releases
   per year, as already happened in the past. That would be mid of March,
   mid of August and mid of December, with code-freeze/feature freeze at
   the beginning of the aforementioned months.

   Contributors of new features should also write a changelog item. This
   will become enforced by a bot, which octo added and enabled during
   the event.

 * There was also a brief discussion over the lack of code reviews or
   a try to understand, why (new) contributors are not doing reviews. The
   general suggestion was to do them anyways, since it helps the project.
   If reviews are hard, then the involved code should get rewritten, since
   it wasn't clearly understandable.

 * For a upcoming future major release, there was the expressed interest
   in having labels for metrics. There is a `design document`_ open up for
   discussion, and we also briefly talked about a migration plan. At the
   end, a collectd 5 client should still be able to cooperate with the
   new version, e.g via the network plugin.

 * A often mentioned feature request is also the (un)loading of plugins
   and with that tightly connected the ability of re-configuring plugins
   at runtime and ideally having a API. This will take some further
   investigation.


.. _collectd: https://collectd.org
.. _FOSDEM: https://fosdem.org/2019/
.. _Camptocamp: https://www.camptocamp.com/
.. _Intel: https://www.intel.com/
.. _`Red Hat`: https://www.redhat.com/
.. _`OPNFV Barometer`: https://wiki.opnfv.org/display/fastpath/Barometer+Home
.. _available: https://wiki.opnfv.org/display/fastpath/Meetings
.. _`code-ownership`: https://github.com/collectd/collectd/issues/2958
.. _`design document`: https://docs.google.com/document/d/173gGP3tUD3yfN2NNHxCv0BsKsacfDlSyoaQIn7MqLtQ
