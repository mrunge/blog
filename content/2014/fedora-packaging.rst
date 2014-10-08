Fedora packaging workshop
#########################
:date: 2014-10-08 14:20
:author: mrunge
:category: Fedora
:tags: Fedora, packaging
:slug: fedora-packaging

On 17th and 18th this month, we'll have a local `Linux conference`_ here. As
part of this, I'll give a short introductory workshop on packaging. The 
following series of posts helps me, to sort things for me and to provide a short
reference for others and myself as well.

Prerequisites
-------------

In order to package and build packages for Fedora and `EPEL`_, you'll 
need a few packages installed

- @development-tools
- @fedora-packager
- rpmlint

You'll install those packages via

:: 

  dnf group install development-tools
  dnf group install fedora-packager
  dnf install rpmlint

You'll find several warnings about not to create packages as root user. Take 
them serious. There's absolutely no reason to build as root user, it's 
plainly risky and dumb to do so.

To separate your user account from the user building rpms a bit more,
add a new user to your system:

::
  
  useradd makerpm

Finally, your user account should become a member of ``mock`` user group.

::

  usermod -a -G mock makerpm


The whole process is documented on `Fedora Wiki`_.




.. _`Linux conference`: http://lit-ol.de/
.. _`EPEL`: https://fedoraproject.org/wiki/EPEL
.. _`Fedora Wiki`: https://fedoraproject.org/wiki/How_to_create_an_RPM_package
.. _`How_to_create_an_RPM_package`: https://fedoraproject.org/wiki/How_to_create_an_RPM_package
