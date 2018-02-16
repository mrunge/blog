Configure your on DNS and dns-cache
###################################
:date: 2016-09-30 19:05
:author: mrunge
:category: Fedora
:tags: Fedora
:slug: unbound
:Status: draft


In my local environment I'm using lots of vms for different purposes, the
same as a few other devices. I really like a consistent network environment,
so I decided to set up a small name server.

This part here is about setting up your workstation to use unbound and
your local DNS to cache dns queries and to forward the queries to your
local DNS. In that script you might need to change hard coded domain names,
WLAN ID and interface names.

.. code::bash
  #!/bin/sh
  INTERFACE=$1
  if [ "$2" == "down" ]; then
      ( if [ "$1" == "wlp3s0" ]; then
          unbound-control forward_remove mydomain.com
      elif [ "$1" == "tun0" ]; then
          unbound-control forward_remove my-vpn-domain.com
      fi ) || :
  elif [ "$2" == "up" ]; then
      ( if [ "$1" == "wlp3s0" ]; then
          ESSID=$(iwconfig $INTERFACE | grep ESSID | cut -d":" -f2 | \
              sed 's/^[^"]*"\|"[^"]*$//g')
          if [ "$ESSID" == "mywlan" ]; then
              unbound-control forward_add mydomain.com 192.168.1.2
          fi
      elif [ "$1" == "tun0" ]; then
          unbound-control forward_add my-vpn-domain.com 10.10.10.10
      fi ) || :
  fi

Now finally your unbound.conf needs changes:

In server section, you'd need to add:
.. code::

  server:
      ....
      local-zone: "10.in-addr.arpa." transparent
      local-zone: "168.192.in-addr.arpa." transparent
      ....
      private-address: 192.168.1.0/24
      private-address: 10.0.0.0/8
      private-domain: "mydomain.com"
      private-domain: "my-vpn-domain.com"

and at the bottom you might want to add

.. code::

  forward-zone:
      name: "mydomain.com"
      forward-addr: 192.168.1.2

  forward-zone:
      name: "my-vpn-domain.com"
      forward-addr: 10.10.10.10

  forward-zone:
      name: "10.in-addr.arpa"
      forward-addr: 10.10.10.10

  forward-zone:
      name: "168.192.in-addr.arpa"
      forward-addr: 192.168.1.2


With that being changed, you can
systemctl enable dnssec-triggerd
systemctl enable unbound
systemctl start dnssec-triggerd
systemctl start unbound

