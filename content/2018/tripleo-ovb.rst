Testing TripleO on own OpenStack deployment
###########################################
:date: 2018-02-16 09:30
:author: mrunge
:category: OpenStack
:tags: OpenStack
:slug: tripleo-ovb
:Status: published

For some use cases, it's quite useful to test TripleO_ deployments
on a OpenStack powered cloud, rather than using a baremetal system.
The following article will show you, how to do it:

We're going to use `tripleo-quickstart`_ . This also assumes, you
have downloaded your OpenStack openrc.sh handy and stored e.g in your
home directory. I've also created another flavor, named tripleo-bm
with 12 Gigs memory and 6 vcpus. In doubt, just give it more RAM.

.. code:: bash

    git clone https://github.com/openstack/tripleo-quickstart
    git clone https://github.com/openstack/tripleo-quickstart-extras
    cp tripleo-quickstart-extras/config/environments/rdcloud.yml tripleo-quickstart/config/environments/mycloud.yml

This copies the configuration for use of RDO-Cloud over, to be ready for
own and required customization. You'll need to modify the file
``tripleo-quickstart/config/environments/mycloud.yml``

.. code:: yaml

    os_username: "{{ lookup('env','OS_USERNAME') }}"
    os_password: "{{ lookup('env','OS_PASSWORD') }}"
    os_tenant_name: "{{ lookup('env','OS_TENANT_NAME') }}"
    os_auth_url: "{{ lookup('env','OS_AUTH_URL') }}"
    os_region_name: "{{ lookup('env', 'OS_REGION_NAME') }}"

    cloud_name: rdocloud
    latest_guest_image:
        newton: CentOS-7-x86_64-GenericCloud-1711
        ocata: CentOS-7-x86_64-GenericCloud-1711
        master: CentOS-x86_64-GenericCloud-1708

    bmc_flavor: m1.small
    baremetal_flavor: m1.large
    undercloud_flavor: tripleo-bm
    provision_net_cidr: 192.168.24.0/24

    custom_nameserver:
        - 192.168.36.9
        - 192.168.36.1
    undercloud_undercloud_nameservers: "{{ custom_nameserver }}"
    external_net: 'ext-net'
    overcloud_dns_servers: "{{ custom_nameserver }}"
    ntp_server: '192.168.36.1'
    boot_from_volume: true

    overcloud_image_url: http://images.rdoproject.org/{{ release }}/rdo_trunk/{{ dlrn_hash|default(dlrn_hash_tag) }}/overcloud-full.tar
    ipa_image_url: http://images.rdoproject.org/{{ release }}/rdo_trunk/{{ dlrn_hash|default(dlrn_hash_tag) }}/ironic-python-agent.tar
    docker_registry_host: trunk.registry.rdoproject.org
    docker_registry_namespace: "{{ release }}"

    mtu: 1350
    mtu_interface:
        - eth0
        - eth1
        - "{% if network_isolation|default(true)|bool %}eth2{% endif %}"

    undercloud_local_mtu: "{{ mtu }}"
    overcloud_neutron_global_physnet_mtu: "{{ mtu }}"

    run_tripleo_validations: False
    # containerized_overcloud: False


Besides minor things like DNS or ntp, one probably needs to adjust are
flavors: They need to exist in your setup. When in doubt, insert a larger
flavor. Also one needs to configure ``external_net``. It should name
the external network in your OpenStack environment, like returned
by ``openstack network list --external``. The guest images
need to exist, at least the one for master. If you don't have
an image you can fetch it here_. The rest can be kept as it is.

Finally, to get reproducible deployments easier, I created a script ``deploy.sh``
to fire up deployments using my customization.

.. code:: bash

    source ~/openrc.sh
    export ENVIRONMENT="mycloud"
    export CUSTOM_REQUIREMENTS_INSTALL=" ansible-lint"
    rm -rf /var/tmp/tripleo_local
    bash devmode.sh --no-gate --ovb -d -w /var/tmp/tripleo_local


If something happens, please file bugs_

.. _collectd : https://github.com/collectd/collectd
.. _tripleo : http://tripleo.org/
.. _`tripleo-quickstart`: https://github.com/openstack/tripleo-quickstart
.. _here: http://cloud.centos.org/centos/7/images/
.. _bugs: https://bugs.launchpad.net/tripleo
