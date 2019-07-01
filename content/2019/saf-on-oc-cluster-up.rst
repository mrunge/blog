Service Assurance on small OpenShift Cluster
############################################
:date: 2019-07-01 13:40
:author: mrunge
:category: metrics
:tags: metrics, OpenShift
:slug: Service-Assurance-on-ocp
:Status: draft

This article is intended to give an overview on how to test the
`Service Assurance Framework`_ on a small deployment of OpenShift.

I've started with a deployment of RHEL 7.x on a baremetal machine.

After using subscribtion-manager to subscribe and attach, I made sure
to have the necessary repositories enabled


.. code:: bash

    subscribtion-manager repos enable amq-interconnect-1-for-rhel-7-server-rpms \
       rhel-7-server-extras-rpms rhel-7-server-ose-3.11-rpms \
       rhel-7-server-optional-rpms rhel-7-server-openstack-14-rpms \
       rhel-7-server-openstack-14-optools-rpms rhel-7-server-rpms


As usual, do a ``yum -y update``.

oc cluster needs to take full control on iptables rules. Thus disable
firewalld, just in case it is running


.. code:: bash

    systemctl disable firewalld


Also install docker:

.. code:: bash

    yum install docker


Now it is time to change the default docker network. Edit `/etc/docker/daemon.json`
and make it read:


.. code::

    {
      "bip":"172.30.0.1/24"
    }

Now it's time to enable dcoker daemon

.. code:: bash

    systemctl ensable docker


Now please reboot your machine and then run

.. code:: bash

    oc cluster up --public-hostname=$(hostname)

That should finish successfully after a short time. Then, if you haven't done so far


.. code:: bash

    yum -y install git
    git clone https://github.com/redhat-service-assurance/telemetry-framework.git

Change into ``telemetry-framework/deploy``. You'll need a token in order to
get access to downstream containers. Log into the `container registry`_ and create
a token. Dowload and save the secret in a file ``serviceassurance-secret.yaml``.
Finally execute run:

.. code:: bash

    oc login -u system:admin
    oc new-project sa-telemetry
    oc create -f ~/telemetry-framework/deploy/serviceassurance-secret.yaml
    openssl req -new -x509 -batch -nodes -days 11000 \
        -subj "/O=io.interconnectedcloud/CN=qdr-white.sa-telemetry.svc.cluster.local" \
        -out qdr-server-certs/tls.crt \
        -keyout qdr-server-certs/tls.key
    oc create secret tls qdr-white-cert --cert=qdr-server-certs/tls.crt --key=qdr-server-certs/tls.key

    ansible-playbook \
    -e "registry_path=$(oc registry info)" \
    -e "imagestream_namespace=$(oc project --short)" \
    deploy_builder.yml

    # need to patch a node in order to allow the current version of the SGO to deploy a SG
    oc patch node localhost -p '{"metadata":{"labels":{"application": "sa-telemetry", "node": "white"}}}'

    ./import-downstream.sh

Once the containers are imported, run

.. code:: bash

    deploy.sh

Press ctrl+c once the prometheus-operator is marked as running.


.. _`Service Assurance Framework`: https://telemetry-framework.readthedocs.io/en/master/
.. _`container registry`: https://access.redhat.com/terms-based-registry/#/accounts
