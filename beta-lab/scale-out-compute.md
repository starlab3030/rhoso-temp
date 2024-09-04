# 노드 확장

## 1. 가상머신 생성

### 1.1 가상머신 디스크 이미지 생성

```bash
cd /var/lib/libvirt/images/
qemu-img create -f qcow2 /var/lib/libvirt/images/
qemu-img info osp-compute-1.qcow2
```

출력 결과
```
[root@hypervisor ~]# cd /var/lib/libvirt/images/

[root@hypervisor images]# qemu-img create -f qcow2 /var/lib/libvirt/images/osp-compute-1.qcow2 150G
Formatting '/var/lib/libvirt/images/osp-compute-1.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=161061273600 lazy_refcounts=off refcount_bits=16

[root@hypervisor images]# qemu-img info osp-compute-1.qcow2
image: osp-compute-1.qcow2
file format: qcow2
virtual size: 150 GiB (161061273600 bytes)
disk size: 196 KiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false

[root@hypervisor images]#
```

### 1.2 가상머신 생성

```bash
virt-install --virt-type kvm --ram 6144 --vcpus 2 --cpu=host-passthrough --os-variant rhel8.4 --disk path=/var/lib/libvirt/images/osp-compute-1.qcow2,device=disk,bus=virtio,format=qcow2 --network network:ocp4-provisioning,mac="de:ad:be:ef:00:07" --network network:ocp4-net --boot hd,network --noautoconsole --vnc --name osp-compute1 --noreboot
virsh start osp-compute1
```

출력 결과
```
[root@hypervisor images]# virt-install --virt-type kvm --ram 6144 --vcpus 2 --cpu=host-passthrough --os-variant rhel8.4 --disk path=/var/lib/libvirt/images/osp-compute-1.qcow2,device=disk,bus=virtio,format=qcow2 --network network:ocp4-provisioning,mac="de:ad:be:ef:00:07" --network network:ocp4-net --boot hd,network --noautoconsole --vnc --name osp-compute1 --noreboot

Starting install...
Domain creation completed.
You can restart your domain by running:
  virsh --connect qemu:///system start osp-compute1

[root@hypervisor images]# virsh start osp-compute1
Domain 'osp-compute1' started

[root@hypervisor images]# virsh domifaddr osp-compute1
 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------

[root@hypervisor images]# virsh dominfo osp-compute1
Id:             38
Name:           osp-compute1
UUID:           378c3d6a-91ba-481f-b91d-8a5b532b2928
OS Type:        hvm
State:          running
CPU(s):         2
CPU time:       61.0s
Max memory:     6291456 KiB
Used memory:    6291456 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: selinux
Security DOI:   0
Security label: system_u:system_r:svirt_t:s0:c388,c849 (permissive)

[root@hypervisor images]#
```

## 2. BMC 구성

### 2.1 vbmc 구성을 위한 방화벽 설정

```bash
iptables -A LIBVIRT_INP -p udp --dport 6237 -j ACCEPT
```

출력 결과
```
[root@hypervisor images]# iptables -A LIBVIRT_INP -p udp --dport 6237 -j ACCEPT

[root@hypervisor images]#
```

### 2.2 osp-compute1를 vbmc에 추가

```bash
vbmc add --username admin --password redhat --port 6237 --address 192.168.123.1 --libvirt-uri qemu:///system osp-compute1
vbmc start osp-compute1
vbmc list
vbmc show osp-compute1
```

출력 결과
```
[root@hypervisor ~]# vbmc add --username admin --password redhat --port 6237 --address 192.168.123.1 --libvirt-uri qemu:///system osp-compute1

[root@hypervisor ~]# vbmc start osp-compute1

[root@hypervisor ~]# vbmc list
+--------------+---------+---------------+------+
| Domain name  | Status  | Address       | Port |
+--------------+---------+---------------+------+
| ocp4-bastion | running | 192.168.123.1 | 6230 |
| ocp4-master1 | running | 192.168.123.1 | 6231 |
| ocp4-master2 | running | 192.168.123.1 | 6232 |
| ocp4-master3 | running | 192.168.123.1 | 6233 |
| ocp4-worker1 | running | 192.168.123.1 | 6234 |
| ocp4-worker2 | running | 192.168.123.1 | 6235 |
| ocp4-worker3 | running | 192.168.123.1 | 6236 |
| osp-compute1 | running | 192.168.123.1 | 6237 |
+--------------+---------+---------------+------+

[root@hypervisor ~]# vbmc show osp-compute1
+-----------------------+----------------+
| Property              | Value          |
+-----------------------+----------------+
| active                | True           |
| address               | 192.168.123.1  |
| domain_name           | osp-compute1   |
| libvirt_sasl_password | ***            |
| libvirt_sasl_username | None           |
| libvirt_uri           | qemu:///system |
| password              | ***            |
| port                  | 6237           |
| status                | running        |
| username              | admin          |
+-----------------------+----------------+

[root@hypervisor ~]#
```
<br>

## 3. 베어메탈 준비

### 3.1 BMC 설정 준비

```bash
git clone https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git labrepo2
yq -y '.' $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml
yq '.spec' $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: osp-compute1-bmc-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cmVkaGF0
---
apiVersion: metal3.io/v1alpha1
kind: BareMetalHost
namespace: openshift-machine-api
metadata:
  name: osp-compute1
labels:
  app: openstack
  workload: compute
spec:
  online: false
  bootMACAddress: de:ad:be:ef:00:07
  bmc:
    address: ipmi://192.168.123.1:6237
    credentialsName: osp-compute1-bmc-secret
  rootDeviceHints:
    deviceName: /dev/vda
```

출력 결과
```
[root@ocp4-bastion ~]# git clone https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git labrepo2
Cloning into 'labrepo2'...
remote: Enumerating objects: 694, done.
remote: Counting objects: 100% (198/198), done.
remote: Compressing objects: 100% (111/111), done.
remote: Total 694 (delta 97), reused 155 (delta 63), pack-reused 496 (from 1)
Receiving objects: 100% (694/694), 5.48 MiB | 12.24 MiB/s, done.
Resolving deltas: 100% (313/313), done.

[root@ocp4-bastion ~]# yq -y '.' $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml

...<snip>...

[root@ocp4-bastion ~]# yq '.spec' $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml
null
{
  "online": false,
  "bootMACAddress": "de:ad:be:ef:00:07",
  "bmc": {
    "address": "ipmi://192.168.123.1:6237",
    "credentialsName": "osp-compute1-bmc-secret"
  },
  "rootDeviceHints": {
    "deviceName": "/dev/vda"
  }
}

[root@ocp4-bastion ~]#
```

### 3.2 베어메탈 호스트 준비

```bash
oc apply -f $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml -n openshift-machine-api
oc get secret/osp-compute1-bmc-secret -n openshift-machine-api
oc get baremetalhost -n openshift-machine-api
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo2/content/files/osp-ng-osp-compute1-bmh.yaml -n openshift-machine-api
secret/osp-compute1-bmc-secret created
baremetalhost.metal3.io/osp-compute1 created

[root@ocp4-bastion ~]# oc get secret/osp-compute1-bmc-secret -n openshift-machine-api
NAME                      TYPE     DATA   AGE
osp-compute1-bmc-secret   Opaque   2      79s

[root@ocp4-bastion ~]# oc get baremetalhost -n openshift-machine-api
NAME           STATE                    CONSUMER                   ONLINE   ERROR   AGE
master1        externally provisioned   ocp-xmwm4-master-0         true             2d4h
master2        externally provisioned   ocp-xmwm4-master-1         true             2d4h
master3        externally provisioned   ocp-xmwm4-master-2         true             2d4h
osp-compute1   inspecting                                          false            22s
worker1        provisioned              ocp-xmwm4-worker-0-vq7vz   true             2d4h
worker2        provisioned              ocp-xmwm4-worker-0-s95rc   true             2d4h
worker3        provisioned              ocp-xmwm4-worker-0-hjs7f   true             2d4h

...<snip>...

[root@ocp4-bastion ~]# oc get baremetalhost -n openshift-machine-api
NAME           STATE                    CONSUMER                   ONLINE   ERROR   AGE
master1        externally provisioned   ocp-xmwm4-master-0         true             2d4h
master2        externally provisioned   ocp-xmwm4-master-1         true             2d4h
master3        externally provisioned   ocp-xmwm4-master-2         true             2d4h
osp-compute1   available                                           false            4m46s
worker1        provisioned              ocp-xmwm4-worker-0-vq7vz   true             2d4h
worker2        provisioned              ocp-xmwm4-worker-0-s95rc   true             2d4h
worker3        provisioned              ocp-xmwm4-worker-0-hjs7f   true             2d4h

[root@ocp4-bastion ~]#
```
* osp-compute1 노드의 *STATE* 값이 *Available* 될 때까지 기다림
* 이는 4분 이상 걸림

### 3.3 베어메탈 노드 레이블 설정

CR인 openstackbaremetalset에 사용될 수 있도록 베어메탈 호스트에 **app:openstack**로 레이블 합니다.

```bash
oc label BareMetalHost osp-compute1 -n openshift-machine-api app=openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc label BareMetalHost osp-compute1 -n openshift-machine-api app=openstack
baremetalhost.metal3.io/osp-compute1 labeled

[root@ocp4-bastion ~]# oc get baremetalhost/osp-compute1 -n openshift-machine-api
NAME           STATE       CONSUMER   ONLINE   ERROR   AGE
osp-compute1   available              false            11m

[root@ocp4-bastion ~]# oc get baremetalhost/osp-compute1 -n openshift-machine-api -o json | jq '.metadata.labels'
{
  "app": "openstack"
}

[root@ocp4-bastion ~]#
```
<br>

## 4. 베어메탈 노드 프로비저닝

### 4.1 데이터-플레인 노드 확장을 위한 설정

```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneNodeSet
metadata:
  name: scale-out-provisioned
spec:
  tlsEnabled: false
  env:
    - name: ANSIBLE_FORCE_COLOR
      value: 'True'
    - name: ANSIBLE_VERBOSITY
      value: '2'
  services:
    - bootstrap
    - download-cache
    - configure-network
    - validate-network
    - install-os
    - configure-os
    - ssh-known-hosts
    - run-os
    - reboot-os
    - install-certs
    - ovn
    - neutron-metadata
    - libvirt
    - nova
  networkAttachments:
    - ctlplane
  baremetalSetTemplate:
    bmhLabelSelector:
      app: openstack
    ctlplaneInterface: enp1s0
    cloudUserName: cloud-admin
  nodes:
    edpm-compute-1:
      hostName: edpm-compute-1
      networks:
        - name: ctlplane
          subnetName: subnet1
        - name: internalapi
          subnetName: subnet1
        - name: storage
          subnetName: subnet1
        - name: tenant
          subnetName: subnet1
        - name: external
          subnetName: subnet1
          defaultRoute: true
  nodeTemplate:
    ansibleSSHPrivateKeySecret: dataplane-ansible-ssh-private-key-secret
    ansible:
      ansibleUser: root
      ansibleVars:
        edpm_network_config_template: "---\n{% set mtu_list = [ctlplane_mtu] %}\n\
          {% for network in nodeset_networks %}\n{{ mtu_list.append(lookup('vars',\
          \ networks_lower[network] ~ '_mtu')) }}\n{%- endfor %}\n{% set min_viable_mtu\
          \ = mtu_list | max %}\nnetwork_config:\n- type: ovs_bridge\n  name: {{ neutron_physical_bridge_name\
          \ }}\n  mtu: {{ min_viable_mtu }}\n  use_dhcp: false\n  dns_servers: {{\
          \ ctlplane_dns_nameservers }}\n  domain: {{ dns_search_domains }}\n  addresses:\n\
          \  - ip_netmask: {{ ctlplane_ip }}/{{ ctlplane_cidr }}\n  routes: {{ ctlplane_host_routes\
          \ }}\n  members:\n  - type: interface\n    name: nic1\n    mtu: {{ min_viable_mtu\
          \ }}\n    # force the MAC address of the bridge to this interface\n    primary:\
          \ true\n{% for network in nodeset_networks if network != 'external' %}\n\
          \  - type: vlan\n    mtu: {{ lookup('vars', networks_lower[network] ~ '_mtu')\
          \ }}\n    vlan_id: {{ lookup('vars', networks_lower[network] ~ '_vlan_id')\
          \ }}\n    addresses:\n    - ip_netmask:\n        {{ lookup('vars', networks_lower[network]\
          \ ~ '_ip') }}/{{ lookup('vars', networks_lower[network] ~ '_cidr') }}\n\
          \    routes: {{ lookup('vars', networks_lower[network] ~ '_host_routes')\
          \ }}\n{% endfor %}\n{% if 'external' in nodeset_networks %}\n- type: ovs_bridge\n\
          \  name: br-ex\n  dns_servers: {{ ctlplane_dns_nameservers }}\n  domain:\
          \ {{ dns_search_domains }}\n  use_dhcp: false\n  members:\n  - type: interface\n\
          \    name: nic2\n    mtu: 1500\n    primary: true\n  routes:\n  - ip_netmask:\
          \ 0.0.0.0/0\n    next_hop: {{ external_gateway_ip | default('192.168.123.1')\
          \ }}\n  addresses:\n  - ip_netmask: {{ external_ip }}/{{ external_cidr }}\n\
          {% endif %}\n"
        edpm_network_config_hide_sensitive_logs: false
        edpm_bootstrap_command: "nmcli connection modify \"System enp1s0\" ipv4.never-default\
          \ yes ipv6.never-default yes\nnmcli connection up \"System enp1s0\"\ncurl\
          \ -ko /etc/pki/ca-trust/source/anchors/demosat-ha.infra.demo.redhat.com.ca.crt\
          \  \"https://demosat-ha.infra.demo.redhat.com/pub/katello-server-ca.crt\"\
          \nupdate-ca-trust\nyum install -y \"https://demosat-ha.infra.demo.redhat.com/pub/katello-ca-consumer-latest.noarch.rpm\"\
          \nsubscription-manager register --org=\"Red_Hat_RHDP_Labs\"  --activationkey=\"\
          demosat-smt-b1374-1711111157\" --serverurl=https://demosat-ha.infra.demo.redhat.com:8443/rhsm\
          \ --baseurl=https://demosat-ha.infra.demo.redhat.com/pulp/repos  \n\nsudo\
          \ subscription-manager repos --disable=* \nsubscription-manager repos --enable=rhceph-6-tools-for-rhel-9-x86_64-rpms\
          \ --enable=rhel-9-for-x86_64-baseos-rpms --enable=rhel-9-for-x86_64-appstream-rpms\
          \ --enable=rhel-9-for-x86_64-highavailability-rpms --enable=openstack-dev-preview-for-rhel-9-x86_64-rpms\
          \ --enable=fast-datapath-for-rhel-9-x86_64-rpms\n"
        ctlplane_host_routes: []
        ctlplane_subnet_cidr: 24
        dns_search_domains: aio.example.com
        ctlplane_vlan_id: 1
        ctlplane_mtu: 1500
        external_mtu: 1500
        external_vlan_id: 44
        external_cidr: '24'
        external_host_routes: []
        internalapi_mtu: 1500
        internalapi_vlan_id: 20
        internalapi_cidr: '24'
        internalapi_host_routes: []
        storage_mtu: 1500
        storage_vlan_id: 21
        storage_cidr: '24'
        storage_host_routes: []
        tenant_mtu: 1500
        tenant_vlan_id: 22
        tenant_cidr: '24'
        tenant_host_routes: []
        neutron_physical_bridge_name: br-osp
        neutron_public_interface_name: eth0
        edpm_nodes_validation_validate_controllers_icmp: false
        edpm_nodes_validation_validate_gateway_icmp: false
        gather_facts: false
        enable_debug: false
        edpm_sshd_allowed_ranges:
          - 172.22.0.0/16
        edpm_podman_buildah_login: true
        edpm_container_registry_logins:
          registry.redhat.io:
            6340056|osp-on-ocp-lb1374: eyJhbGciOiJSUzUxMiJ9.eyJzdWIiOiI1Y2EzM2NjNGY4NWM0MmZmYTI3YmU5Y2UyMWI3M2JjMCJ9.GAxgg6Ht2oCS8zxHdwQw9kSD6RHeQOWYaDOcnQB5RElewQKvZmcNWi-YJdInJ5iXTE9r9tGVIN7fhFJL7f-hhL1PK2RVzZHD8qyfkMWcCEF5GUvp8rDX4GDrSkqjpUD44teWYkOy9Nb-3pOGzRIC7qs88uSxMz7hfil4I_HmjF4AAPIi4j3QZhp0lqrXzzf7vt6NLlizDFa2XTcPf_vQqReFu3A_5iWfy8XmLlC7QIixeVv2IE-ahRqM_UDCf5Dg3n2WpYvmP5jcSPFOLoT7sMimyeaPBna793boiX2swmeGHQ23tx1nFavCUavGv_cDRAvzVXCJ2NROTJ5unHiN7CXEbzm4Rg-65tY4D0YynTU8L6t0gYtXYYY9_wi1xNs-cShAmCMh1ySJn9nBcq4ydvH7eQnhSEvoK0bPsN_vWJCgOQBQyOdpTfRMU6piAy9H1zJ0KzsSzuKSS8fX0m9oN7narZPl34DTiEUTDeW8_SS6vJjHr_Q9O_X4mVeeQhH2ocN_4M9R6A89tmQ2jObuWm-cu1Yk-G6FSPUONhsoC_99nQnICS4mAuCWWDHxFY61hIrreVZBSH053MgfSaG2sqTb26MkxKWx-TP1sx18pb1xmo4IQEwILIbLlSPA3vafbrbQO5RQcm3UYKtYwev0vAlL5taXiTuLEyPscdzv0Sc
```

```bash
oc apply -f $HOME/labrepo2/content/files/osp-ng-dataplane-node-set-deploy-scale-out.yaml
oc get openstackdataplanenodeset/scale-out-provisioned -n openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo2/content/files/osp-ng-dataplane-node-set-deploy-scale-out.yaml
openstackdataplanenodeset.dataplane.openstack.org/scale-out-provisioned created

[root@ocp4-bastion ~]# oc get openstackdataplanenodeset/scale-out-provisioned -n openstack
NAME                    STATUS   MESSAGE
scale-out-provisioned   False    Setup started

[root@ocp4-bastion ~]#
```

### 4.2 데이터-플레인 노드 확장

```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneDeployment
metadata:
  name: openstack-scale-out-provisioned
spec:
  nodeSets:
    - scale-out-provisioned
```

```bash
oc apply -f $HOME/labrepo2/content/files/osp-ng-dataplane-deployment-scale-out.yaml
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo2/content/files/osp-ng-dataplane-deployment-scale-out.yaml
openstackdataplanedeployment.dataplane.openstack.org/openstack-scale-out-provisioned created

[root@ocp4-bastion ~]# oc get openstackdataplanedeployment -n openstack
NAME                              NODESETS                    STATUS    MESSAGE
openstack-edpm-ipam               ["openstack-edpm-ipam"]     True      Setup complete
openstack-scale-out-provisioned   ["scale-out-provisioned"]   Unknown   Setup started

[root@ocp4-bastion ~]#
```

### 4.3 데이터-플레인 노드 프로비저닝

```bash
oc get bmh -n openshift-machine-api
oc get bmh/osp-compute1 -n openshift-machine-api
```

출력 결과
```
[root@ocp4-bastion ~]# oc get bmh -n openshift-machine-api
NAME           STATE                    CONSUMER                   ONLINE   ERROR   AGE
master1        externally provisioned   ocp-xmwm4-master-0         true             2d4h
master2        externally provisioned   ocp-xmwm4-master-1         true             2d4h
master3        externally provisioned   ocp-xmwm4-master-2         true             2d4h
osp-compute1   provisioning             scale-out-provisioned      true             30m
worker1        provisioned              ocp-xmwm4-worker-0-vq7vz   true             2d4h
worker2        provisioned              ocp-xmwm4-worker-0-s95rc   true             2d4h
worker3        provisioned              ocp-xmwm4-worker-0-hjs7f   true             2d4h

[root@ocp4-bastion ~]# oc get bmh/osp-compute1 -n openshift-machine-api
NAME           STATE          CONSUMER                ONLINE   ERROR   AGE
osp-compute1   provisioning   scale-out-provisioned   true             31m

[root@ocp4-bastion ~]# oc get bmh/osp-compute1 -n openshift-machine-api
NAME           STATE         CONSUMER                ONLINE   ERROR   AGE
osp-compute1   provisioned   scale-out-provisioned   true             34m

[root@ocp4-bastion ~]#
```
* 노드 프로비저닝 후에 *STATE*가 *provisioned*로 바뀐 것을 확인
* 이 후에는 pre-provisioned와 비슷한 방식으로 설치가 진행됨

### 4.4 설치 및 로그 확인

```bash
oc logs -l app=openstackansibleee -f --max-log-requests 10
oc get openstackdataplanedeployment
oc get openstackdataplanenodeset
```
* 로그 수가 많은 경우에 값을 10, 20 등으로 바꾸어서 확인 

출력 결과
```
[root@ocp4-bastion ~]# oc logs -l app=openstackansibleee -f --max-log-requests 20

...<snip>...

[root@ocp4-bastion ~]# oc get openstackdataplanedeployment
NAME                              NODESETS                    STATUS   MESSAGE
openstack-edpm-ipam               ["openstack-edpm-ipam"]     True     Setup complete
openstack-scale-out-provisioned   ["scale-out-provisioned"]   False    Deployment in progress

[root@ocp4-bastion ~]# oc get openstackdataplanenodeset
NAME                    STATUS   MESSAGE
openstack-edpm-ipam     True     NodeSet Ready
scale-out-provisioned   False    Deployment in progress

...<snip>...

[root@ocp4-bastion ~]# oc get openstackdataplanedeployment
NAME                              NODESETS                    STATUS   MESSAGE
openstack-edpm-ipam               ["openstack-edpm-ipam"]     True     Setup complete
openstack-scale-out-provisioned   ["scale-out-provisioned"]   True     Setup complete

[root@ocp4-bastion ~]# oc get openstackdataplanenodeset
NAME                    STATUS   MESSAGE
openstack-edpm-ipam     True     NodeSet Ready
scale-out-provisioned   True     NodeSet Ready

[root@ocp4-bastion ~]#
```
* 설치에는 시간이 걸림
<br>

## 5. 확장된 노드 확인

### 5.1 새로 확장된 컴퓨트 노드를 컴퓨트 셀에 매핑

```bash
oc rsh nova-cell0-conductor-0 nova-manage cell_v2 discover_hosts --verbose
```

출력 결과
```
[root@ocp4-bastion ~]# oc rsh nova-cell0-conductor-0 nova-manage cell_v2 discover_hosts --verbose
Modules with known eventlet monkey patching issues were imported prior to eventlet monkey patching: urllib3. This warning can usually be ignored if the caller is only importing and not executing nova code.
2024-08-30 10:47:55.187 122139 WARNING oslo_policy.policy [None req-277c038a-6d8d-4742-b4d8-c8f284606824 - - - - - -] JSON formatted policy_file support is deprecated since Victoria release. You need to use YAML format which will be default in future. You can use ``oslopolicy-convert-json-to-yaml`` tool to convert existing JSON-formatted policy file to YAML-formatted in backward compatible way: https://docs.openstack.org/oslo.policy/latest/cli/oslopolicy-convert-json-to-yaml.html.
2024-08-30 10:47:55.187 122139 WARNING oslo_policy.policy [None req-277c038a-6d8d-4742-b4d8-c8f284606824 - - - - - -] JSON formatted policy_file support is deprecated since Victoria release. You need to use YAML format which will be default in future. You can use ``oslopolicy-convert-json-to-yaml`` tool to convert existing JSON-formatted policy file to YAML-formatted in backward compatible way: https://docs.openstack.org/oslo.policy/latest/cli/oslopolicy-convert-json-to-yaml.html.
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 46031296-eae2-4d8c-ac02-47e546f42325
Checking host mapping for compute host 'edpm-compute-1.ctlplane.aio.example.com': 557ef744-54d4-445e-a711-b3b033073ddb
Creating host mapping for compute host 'edpm-compute-1.ctlplane.aio.example.com': 557ef744-54d4-445e-a711-b3b033073ddb
Found 1 unmapped computes in cell: 46031296-eae2-4d8c-ac02-47e546f42325

[root@ocp4-bastion ~]#
```

### 5.2 컴퓨프 서비스 확인을 통해 노드 확인

```bash
oc rsh -n openstack openstackclient
openstack compute service list
```

출력 결과
```
[root@ocp4-bastion ~]# oc rsh -n openstack openstackclient

sh-5.1$ openstack compute service list
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                                    | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+
| a1fe4d67-acc3-459c-9e47-1a642b844fe2 | nova-conductor | nova-cell0-conductor-0                  | internal | enabled | up    | 2024-08-30T10:50:57.000000 |
| 3e50fc7d-f358-4e9e-ac1c-69b07efd8f52 | nova-scheduler | nova-scheduler-0                        | internal | enabled | up    | 2024-08-30T10:50:57.000000 |
| 7109dd83-3d66-4107-a1d7-baf7653cf6e1 | nova-conductor | nova-cell1-conductor-0                  | internal | enabled | up    | 2024-08-30T10:50:56.000000 |
| 124b4605-906f-4e1b-8c7c-5ec4318fa8f7 | nova-compute   | edpm-compute-0.ctlplane.aio.example.com | nova     | enabled | up    | 2024-08-30T10:50:56.000000 |
| cb5a88dd-09c6-4037-84f1-480924e914f4 | nova-compute   | edpm-compute-1.ctlplane.aio.example.com | nova     | enabled | up    | 2024-08-30T10:50:57.000000 |
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+

sh-5.1$ exit

sh-5.1$
```
* edpm-compute-1가 리스트에 있음을 확인

### 5.3 네트워크 확인

#### 5.3.1 *openstack* 네임스페이스에서 *ipset* 확인

```bash
oc get ipset -n openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc get ipset -n openstack
NAME             READY   MESSAGE          RESERVATION
edpm-compute-0   True    Setup complete
edpm-compute-1   True    Setup complete

[root@ocp4-bastion ~]#
```

#### 5.3.2 edpm-compute-1 정보 확인
```bash
oc describe ipset edpm-compute-1 -n openstack
```

```yaml
Name:         edpm-compute-1
Namespace:    openstack
Labels:       <none>
Annotations:  <none>
API Version:  network.openstack.org/v1beta1
Kind:         IPSet
Metadata:
  Creation Timestamp:  2024-08-30T10:18:42Z
  Finalizers:
    IPSet
  Generation:  1
  Owner References:
    API Version:           dataplane.openstack.org/v1beta1
    Block Owner Deletion:  true
    Controller:            true
    Kind:                  OpenStackDataPlaneNodeSet
    Name:                  scale-out-provisioned
    UID:                   129a272f-2ab0-4bbf-bcf4-7cda5d9c6135
  Resource Version:        2872972
  UID:                     a4117ff2-21c3-44de-9fa1-ebb953653a5e
Spec:
  Immutable:  false
  Networks:
    Name:           ctlplane
    Subnet Name:    subnet1
    Name:           internalapi
    Subnet Name:    subnet1
    Name:           storage
    Subnet Name:    subnet1
    Name:           tenant
    Subnet Name:    subnet1
    Default Route:  true
    Name:           external
    Subnet Name:    subnet1
Status:
  Conditions:
    Last Transition Time:  2024-08-30T10:18:42Z
    Message:               Setup complete
    Reason:                Ready
    Status:                True
    Type:                  Ready
    Last Transition Time:  2024-08-30T10:18:42Z
    Message:               Input data complete
    Reason:                Ready
    Status:                True
    Type:                  InputReady
    Last Transition Time:  2024-08-30T10:18:42Z
    Message:               Reservation successful
    Reason:                Ready
    Status:                True
    Type:                  ReservationReady
  Observed Generation:     1
  Reservations:
    Address:     172.22.0.101
    Cidr:        172.22.0.0/24
    Dns Domain:  ctlplane.aio.example.com
    Gateway:     172.22.0.1
    Mtu:         1500
    Network:     ctlplane
    Subnet:      subnet1
    Address:     192.168.123.62
    Cidr:        192.168.123.0/24
    Dns Domain:  external.aio.example.com
    Gateway:     192.168.123.1
    Mtu:         1500
    Network:     external
    Routes:
      Destination:  0.0.0.0/0
      Nexthop:      192.168.123.1
    Subnet:         subnet1
    Address:        172.17.0.101
    Cidr:           172.17.0.0/24
    Dns Domain:     internalapi.aio.example.com
    Mtu:            1500
    Network:        internalapi
    Subnet:         subnet1
    Vlan:           20
    Address:        172.18.0.101
    Cidr:           172.18.0.0/24
    Dns Domain:     storage.aio.example.com
    Mtu:            1500
    Network:        storage
    Subnet:         subnet1
    Vlan:           21
    Address:        172.19.0.101
    Cidr:           172.19.0.0/24
    Dns Domain:     tenant.aio.example.com
    Mtu:            1500
    Network:        tenant
    Subnet:         subnet1
    Vlan:           22
Events:             <none>
```
<br>


#### 5.3.3 *edpm-compute-1* 노드의 *ctlplane* 확인

```bash
oc get ipset/edpm-compute-1 -n openstack -o json | jq '.status.reservations[]|select(.network|match("ctlplane"))'
```

```json
{
  "address": "172.22.0.101",
  "cidr": "172.22.0.0/24",
  "dnsDomain": "ctlplane.aio.example.com",
  "gateway": "172.22.0.1",
  "mtu": 1500,
  "network": "ctlplane",
  "subnet": "subnet1"
}
```

### 5.4 확장된 노드 연결

```bash
ssh -i /root/.ssh/id_rsa_compute cloud-admin@172.22.0.101
ip --br a s
```

출력 결과
```
[root@ocp4-bastion ~]# ssh -i /root/.ssh/id_rsa_compute cloud-admin@172.22.0.101
The authenticity of host '172.22.0.101 (172.22.0.101)' can't be established.
ED25519 key fingerprint is SHA256:drs0mtN3xeqIs/lSlDOw2Je57oIvKTazafB2/vhz7iU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.22.0.101' (ED25519) to the list of known hosts.
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Fri Aug 30 10:44:17 2024 from 172.22.0.30

[cloud-admin@edpm-compute-1 ~]$ ip --br a s
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp1s0           UP             fe80::dcad:beff:feef:7/64
enp2s0           UP             fe80::5054:ff:fef0:50c9/64
ovs-system       DOWN
br-osp           UNKNOWN        172.22.0.101/24 fe80::dcad:beff:feef:7/64
vlan22           UNKNOWN        172.19.0.101/24 fe80::a41e:44ff:feb8:e11c/64
vlan20           UNKNOWN        172.17.0.101/24 fe80::9845:e4ff:fe91:fec7/64
vlan21           UNKNOWN        172.18.0.101/24 fe80::f87b:aeff:fe2c:c562/64
br-ex            UNKNOWN        192.168.123.62/24 fe80::5054:ff:fef0:50c9/64
br-int           DOWN
genev_sys_6081   UNKNOWN        fe80::78b8:60ff:fe34:2ba8/64

[cloud-admin@edpm-compute-1 ~]$
```
<br>

## 마무리

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 접속 <<](./access-openstack.md)