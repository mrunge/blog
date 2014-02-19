OpenStack Summit Hong Kong
##########################
:date: 2013-11-12 19:54
:author: mrunge
:category: OpenStack
:slug: openstack-summit-hong-kong

From my very personal view on the past OpenStack summit in Hong Kong, it
has been a good and productive summit. Luckily, both of my sessions were
accepted:

Separate OpenStack Dashboard and Horizon
----------------------------------------

The first session was about how to separate Horizon and OpenStack
Dashboard. It is very little known to folks outside the Horizon project,
how both parts relate to each other.

Just looking at Horizon's source code, one will see mainly two
directories, horizon and openstack\_dashboard. The subject is, to
distribute code from both directories in two separate projects. The
directory horizon contains the framework, which is used to build the
WebUI provided in the directory openstack\_dashboard. The latter is
mostly known as Horizon. Mixing the terms is quite confusing for anyone.
For the rest, I'll use the terms WebUI and framework for OpenStack
Dashboard resp. for the code provided in the horizon dir.

In the past, most changes on the source code have been made in only one
of both parts, where the majority of changes were improvements at the
WebUI part. The idea behind dividing both is, that we'll be able to
promote the framework separate from OpenStack at all, and it might
attract new developers or users for this. The framework fills gaps in
the Django, and thus will be a valuable addition directly to Django or
to the Django eco-system outside the OpenStack context. In
distributions, like in Ubuntu or Fedora, both parts are separated for a
long time or have been divided since ever. When using a different WebUI,
there will be no need to handle dependencies for the unused WebUI part
at all.

The main question here, if we shall do this or not is simply: is the
framework mature and stable enough. Since changes in the framework
itself will affect two repositories instead of one, this is the key
question. Attendants of the session agreed, it's a good idea to split
both parts, and I'm planning to do this during the next development
cycle.

Plugin architecture for OpenStack dashboard
-------------------------------------------

This session was more technical than the other. Currently, when adding
an additional dashboard, tab, etc. to the WebUI, it's required to change
settings.py each time. This is e.g not update safe and not packaging
friendly at all. The outcome of the session was, we (== Horizon
developers) should investigate stevedore, which is used by other
OpenStack projects for the same reasons. The same should work with a bit
of hacking for auto-discovery for Panels and PanelGroups. This is
something on my personal todo list as well, and luckily Monty Taylor
offered his help for this topic as well.
