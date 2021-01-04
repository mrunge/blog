Title: Kubernetes on Raspberry Pi 4 on Fedora
Date: 2021-01-04 11:00
Author: mrunge
Category: Fedora
Tags: Fedora, k8s, kubernetes
Slug: k8s-raspi4-fedora
Status: published

Recently, I bought a couple of Raspberry Pi 4, one with 4 GB and 2
equipped with 8 GB of RAM. When I bought the first one, there was no
option to get bigger memory. However, I saw this as a game and thought
to give this a try. I also bought SSDs for these and USB3 to SATA
adapters. Before purchasing anything, you may want to take a look at
[James Archers page](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/).
Unfortunately, there are a couple on adapters on the marked, which
don't work that well.

## Deploying Fedora 33 Server

Initially, I followed the [description](https://fwmotion.com/blog/operating-systems/2020-09-04-installing-fedora-server-onto-pi4/)
to deploy Fedora 32; it works the same way for Fedora 33 Server (in my case here).

Because ceph requires a partition (or better: a whole disk), I used the
traditional setup using partitions and no LVM.


## Deploying Kubernetes


```bash
git clone https://github.com/kubernetes-sigs/kubespray
cd kubespray
```

I followed the documentation and created an inventory. For the container
runtime, I picked `crio`, and as `calico` as network plugin.

Because of [an issue](https://github.com/projectcalico/calico/issues/4256),
I had to patch `roles/download/defaults/main.yml`:

    diff --git a/roles/download/defaults/main.yml b/roles/download/defaults/main.yml
    index a97be5a6..d4abb341 100644
    --- a/roles/download/defaults/main.yml
    +++ b/roles/download/defaults/main.yml
    @@ -64,7 +64,7 @@ quay_image_repo: "quay.io"

     # TODO(mattymo): Move calico versions to roles/network_plugins/calico/defaults
     # after migration to container download
    -calico_version: "v3.16.5"
    +calico_version: "v3.15.2"
     calico_ctl_version: "{{ calico_version }}"
     calico_cni_version: "{{ calico_version }}"
     calico_policy_version: "{{ calico_version }}"
    @@ -520,13 +520,13 @@ etcd_image_tag: "{{ etcd_version }}{%- if image_arch != 'amd64' -%}-{{ image_arc
     flannel_image_repo: "{{ quay_image_repo }}/coreos/flannel"
     flannel_image_tag: "{{ flannel_version }}"
     calico_node_image_repo: "{{ quay_image_repo }}/calico/node"
    -calico_node_image_tag: "{{ calico_version }}"
    +calico_node_image_tag: "{{ calico_version }}-arm64"
     calico_cni_image_repo: "{{ quay_image_repo }}/calico/cni"
    -calico_cni_image_tag: "{{ calico_cni_version }}"
    +calico_cni_image_tag: "{{ calico_cni_version }}-arm64"
     calico_policy_image_repo: "{{ quay_image_repo }}/calico/kube-controllers"
    -calico_policy_image_tag: "{{ calico_policy_version }}"
    +calico_policy_image_tag: "{{ calico_policy_version }}-arm64"
     calico_typha_image_repo: "{{ quay_image_repo }}/calico/typha"
    -calico_typha_image_tag: "{{ calico_typha_version }}"
    +calico_typha_image_tag: "{{ calico_typha_version }}-arm64"
     pod_infra_image_repo: "{{ kube_image_repo }}/pause"
     pod_infra_image_tag: "{{ pod_infra_version }}"
     install_socat_image_repo: "{{ docker_image_repo }}/xueshanf/install-socat"


## Deploy Ceph


Ceph requires a raw partition. Make sure, you have an empty partition available.

```bash
[root@node1 ~]# lsblk -f
NAME FSTYPE FSVER LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda
├─sda1
│    vfat   FAT32 UEFI  7DC7-A592
├─sda2
│    vfat   FAT32       CB75-24A9                               567.9M     1% /boot/efi
├─sda3
│    xfs                cab851cb-1910-453b-ae98-f6a2abc7f0e0    804.7M    23% /boot
├─sda4
│
├─sda5
│    xfs                6618a668-f165-48cc-9441-98f4e2cc0340     27.6G    45% /
└─sda6
```

In my case, there are `sda4` and `sda6` not formatted. `sda4` is very
small and will be ignored, `sda6` will be used.

Using rook is pretty straightforward

```bash
git clone --single-branch --branch v1.5.4 https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
kubectl create -f cluster.yaml
```



