Horizon topics during Kilo development summit
#############################################
:date: 2014-11-10 14:45
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: horizon-kilo-summit


The past Kilo development summit had the following things for Horizon covered.

Horizon Operators, Deployers, and Users
---------------------------------------
The first session was a `meetup between Horizon developers and operators`_.
Most named painpoint was: managing a huge number of users and resources.
Missing pagination on all kinds of pages was mentioned a few times. The
other issue seems to be, that region switching requires too many manual steps
or sometimes behaves not as expected. Overall that should be improved in terms
of user experience.

The last issue mentioned was: error messages often don't really help. There's
a `blueprint to improve error messaging`_. Developers understand, that this is
an issue, but sadly it's not going to be solved that easily.


Keystone-Horizon-CLI/Federation/SSO
-----------------------------------

A `Keystone-Horizon cross-project session`_ was held to discuss issues around
integrating federated authentication and single sign in for Keystone and
Horizon.

Django-Angular Playing Nice
---------------------------

Thai Tran presented his ideas on making `Django-Angular Playing Nice`_.
Basic idea is to slowly replace Django parts by using pure
JavaScript and AngularJS and have just a tiny core proxying JSON-data from
underlying services. That'd be the least obstructive way to
re-implement everything in using JavaScript and AngularJS. This was
already proposed at the last summit; since then, not much has happened yet.



Re-Implementation in JavaScript using AngularJS
-----------------------------------------------

Richard Jones from Rackspace presented his `demo`_ on re-implementing Horizon
nearly purely in JavaScript.
There were quite a few people in the audience, apparently doing more or less
the same. Unfortunately, they didn't chose to share their code or to propse
that code upstream.


Horizon-Contributors meetup
---------------------------

We discussed a `lot topics`_. Sadly, we shared a the same room with
keystone contributors and it was quite noisy.

JavaScript
==========

Again, using a more JavaScript-centric approach for Horizon was discussed.
AngularJS was chosen as the framework to be used. The navigation
system and panels will stay in Django. Content panel is left up to the
implementer and may be Angular.

There wasn't a real agreement on how to start. Since it's not entirely worked
out yet, one might need to experiment here. One proposal is, to start
simple; the other direction was, to start with the most complex piece,
"Start-Instance-Workflow". The latter might be a big complex to land during
the next six months. Launch instance is known as a big issue though.

We agreed, we'll need documentation, tutorials for new contributors, and 
review guidelines. Richard Jones will follow-up on the OpenStack Developers
mailing list.


UX
==

A new Launch instance design was proposed. We briefly discussed adding
images for styling glance images. 


Repo-Split
==========

This is a longer going effort and required e.g unbundling embedded copies
of JavaScript libraries (which is good anyways). We will need to freeze the
code for a limited time. There are `concerns about naming`_ the separate
parts. Aside from breaking dependent packages we will certainly confuse
contributors and users with changing the name of ``openstack-dashboard`` to
``horizon`` and to rename former ``horizon`` to something else.

Kilo priorities
===============


There are quite a few todo items left from the operators session, like:

- error message improvement
- filtering, search, and pagination in tables is still an issue
- documentation should become clearer, there are still topics not entierely
  covered in docs.
- inclusion of AngularJS will be a big task, views will be rewritten piece
  by piece
- Repo split
- blueprints will have to use a mandatory template in the future
- Ironic is now integrated in OpenStack, but currently does not have any
  representation in Horizon


.. _`demo`: https://github.com/r1chardj0n3s/angboard
.. _`meetup between Horizon developers and operators`:
 https://etherpad.openstack.org/p/kilo-horizon-operators-users
.. _`blueprint to improve error messaging`:
 https://blueprints.launchpad.net/horizon/+spec/improve-error-message-details-for-usability
.. _`Keystone-Horizon cross-project session`:
 https://etherpad.openstack.org/p/kilo-keystone-horizon-cli-federation-sso
.. _`Django-Angular Playing Nice`:
 https://etherpad.openstack.org/p/kilo-horizon-django-angular-playing-nice
.. _`lot topics`: https://etherpad.openstack.org/p/kilo-horizon-contributors-meetup
.. _`concerns about naming`: http://lists.openstack.org/pipermail/openstack-dev/2014-November/050043.html
