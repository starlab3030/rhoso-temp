# 오픈스택 데이터-플레인 구성

1. [데이터-플레인 네트워크](./configure-data-plane.md#1-데이터-플레인-네트워크)<br>
2. [컴퓨트 노드 준비](./configure-data-plane.md#2-컴퓨트-노드-준비)<br>
   2.1 [가상머신 이미지 템플릿 준비](./configure-data-plane.md#21-가상머신-이미지-템플릿-준비)<br>
   2.2 [가상머신 이미지 템플릿 구성](./configure-data-plane.md#22-가상머신-이미지-템플릿-구성)<br>
   2.3 [컴퓨트 노드 생성](./configure-data-plane.md#23-컴퓨트-노드-생성)<br>
   2.4 [osp-compute0 리포지토리 구성](./configure-data-plane.md#24-osp-compute0-리포지토리-구성)<br>
   2.5 [필요한 패키지 구성](./configure-data-plane.md#25-필요한-패키지-설치)<br>
   2.6 [osp-compute0 가상머신 스냅샷 생성](./configure-data-plane.md#26-osp-compute0-가상머신-스냅샷-생성)<br>
   2.7 [osp-compute0에 SSH 키 복사](./configure-data-plane.md#27-osp-compute0에-ssh-키-복사)<br>
3. [데이터-플레인 설정 끝내기](./configure-data-plane.md#3-데이터-플레인-설정-끝내기)<br>
   3.1 [시크릿 생성](./configure-data-plane.md#31-시크릿-생성)<br>
   3.2 [데이터-플레인 배포](./configure-data-plane.md#32-데이터-플레인-배포)<br>
4. [요약](./configure-data-plane.md#요약)<br>
<br>

## 1. 데이터-플레인 네트워크

### 1.1 데이터-플레인 네트워크 구성 확인

```bash
yq -y '.' $HOME/labrepo/content/files/osp-ng-dataplane-netconfig.yaml
```

데이터-플레인 네트워크 구성하는 YAML 파일
```yaml
apiVersion: network.openstack.org/v1beta1
kind: NetConfig
metadata:
  name: openstacknetconfig
  namespace: openstack
spec:
  networks:
    - name: ctlplane
      dnsDomain: ctlplane.aio.example.com
      subnets:
        - name: subnet1
          allocationRanges:
            - end: 172.22.0.120
              start: 172.22.0.100
            - end: 172.22.0.200
              start: 172.22.0.150
          cidr: 172.22.0.0/24
          gateway: 172.22.0.1
    - name: internalapi
      dnsDomain: internalapi.aio.example.com
      subnets:
        - name: subnet1
          allocationRanges:
            - end: 172.17.0.250
              start: 172.17.0.100
          excludeAddresses:
            - 172.17.0.10
            - 172.17.0.12
          cidr: 172.17.0.0/24
          vlan: 20
    - name: tenant
      dnsDomain: tenant.aio.example.com
      subnets:
        - name: subnet1
          allocationRanges:
            - end: 172.19.0.250
              start: 172.19.0.100
          excludeAddresses:
            - 172.19.0.10
            - 172.19.0.12
          cidr: 172.19.0.0/24
          vlan: 22
    - name: storage
      dnsDomain: storage.aio.example.com
      subnets:
        - name: subnet1
          allocationRanges:
            - end: 172.18.0.250
              start: 172.18.0.100
          excludeAddresses:
            - 172.18.0.10
            - 172.18.0.12
          cidr: 172.18.0.0/24
          vlan: 21
    - name: external
      dnsDomain: external.aio.example.com
      subnets:
        - name: subnet1
          allocationRanges:
            - end: 192.168.123.90
              start: 192.168.123.61
          cidr: 192.168.123.0/24
          gateway: 192.168.123.1
```
<br>

### 1.2 데이터-플레인 네트워크 적용

데이터-플레인 네트워크 구성 준비
```bash
oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-netconfig.yaml
oc get netconfig -n openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-netconfig.yaml
netconfig.network.openstack.org/openstacknetconfig created

[root@ocp4-bastion ~]# oc get netconfig -n openstack
NAME                 AGE
openstacknetconfig   3s

[root@ocp4-bastion ~]#
```
<br>

## 2. 컴퓨트 노드 준비

### 2.1 가상머신 이미지 템플릿 준비

기존 생성된 RHEL9 게스트 이미지 기반 오픈스택 가상머신 이미지 템플릿 준비
```bash
sudo -i
cd /var/lib/libvirt/images/
cp rhel-9.4-x86_64-kvm.qcow2 rhel9-guest.qcow2
qemu-img info rhel9-guest.qcow2
qemu-img resize rhel9-guest.qcow2 +90G
qemu-img info rhel9-guest.qcow2
chown -R qemu:qemu rhel9-guest.qcow2
ls -lh rhel9-guest.qcow2
```

출력 결과
```
[lab-user@hypervisor ~]$ sudo -i

[root@hypervisor ~]# cd /var/lib/libvirt/images/

[root@hypervisor images]# cp rhel-9.4-x86_64-kvm.qcow2 rhel9-guest.qcow2

[root@hypervisor images]# qemu-img info rhel9-guest.qcow2
image: rhel9-guest.qcow2
file format: qcow2
virtual size: 10 GiB (10737418240 bytes)  <<<< 가상 디스크 이미지 크기: 10 GiB
disk size: 913 MiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false

[root@hypervisor images]# qemu-img resize rhel9-guest.qcow2 +90G
Image resized.

[root@hypervisor images]# qemu-img info rhel9-guest.qcow2
image: rhel9-guest.qcow2
file format: qcow2
virtual size: 100 GiB (107374182400 bytes) <<<< 가상 디스크 이미지 크기: 100 GiB
disk size: 913 MiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false

[root@hypervisor images]# chown -R qemu:qemu rhel9-guest.qcow2

[root@hypervisor images]# ls -lh rhel9-guest.qcow2
-rw-r--r--. 1 qemu qemu 913M Aug 29 20:53 rhel9-guest.qcow2

[root@hypervisor images]#
```
<br>

### 2.2 가상머신 이미지 템플릿 구성

오픈스택 가상머신 이미지 템플릿 구성 
```bash
virt-customize -a rhel9-guest.qcow2 --run-command 'growpart /dev/sda 4'
virt-customize -a rhel9-guest.qcow2 --run-command 'xfs_growfs /'
virt-customize -a rhel9-guest.qcow2 --root-password password:redhat
virt-customize -a rhel9-guest.qcow2 --run-command 'systemctl disable cloud-init'
virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --ssh-inject root:file:/root/.ssh/id_rsa.pub
virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --selinux-relabel
```

출력 결과
```
[root@hypervisor images]# virt-customize -a rhel9-guest.qcow2 --run-command 'growpart /dev/sda 4'
[   0.0] Examining the guest ...
[   9.4] Setting a random seed
[   9.5] Setting the machine ID in /etc/machine-id
[   9.5] Running: growpart /dev/sda 4
[  10.0] Finishing off

[root@hypervisor images]# virt-customize -a rhel9-guest.qcow2 --run-command 'xfs_growfs /'
[   0.0] Examining the guest ...
[   6.0] Setting a random seed
[   6.0] Running: xfs_growfs /
[   6.1] Finishing off

[root@hypervisor images]# virt-customize -a rhel9-guest.qcow2 --root-password password:redhat
[   0.0] Examining the guest ...
[   5.9] Setting a random seed
[   6.0] Setting passwords
[   7.3] Finishing off

[root@hypervisor images]# virt-customize -a rhel9-guest.qcow2 --run-command 'systemctl disable cloud-init'
[   0.0] Examining the guest ...
[   5.8] Setting a random seed
[   5.8] Running: systemctl disable cloud-init
[   6.0] Finishing off

[root@hypervisor images]# virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --ssh-inject root:file:/root/.ssh/id_rsa.pub
[   0.0] Examining the guest ...
[   5.8] Setting a random seed
[   5.8] SSH key inject: root
[   7.1] Finishing off

[root@hypervisor images]# virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --selinux-relabel
[   0.0] Examining the guest ...
[   6.0] Setting a random seed
[   6.0] SELinux relabelling
[  19.6] Finishing off

[root@hypervisor images]#
```
<br>

### 2.3 컴퓨트 노드 생성

#### 2.3.1 osp-compute0 가상머신 이미지 복제

오픈스택 컴퓨트 노드 (osp-compute0) 가상머신 이미지 준비
```bash
qemu-img create -f qcow2 -F qcow2 -b /var/lib/libvirt/images/rhel9-guest.qcow2 /var/lib/libvirt/images/osp-compute-0.qcow2
ls -lh /var/lib/libvirt/images/osp-compute-0.qcow2
qemu-img info /var/lib/libvirt/images/osp-compute-0.qcow2
```

출력 결과
```
[root@hypervisor ~]# qemu-img create -f qcow2 -F qcow2 -b /var/lib/libvirt/images/rhel9-guest.qcow2 /var/lib/libvirt/images/osp-compute-0.qcow2
Formatting '/var/lib/libvirt/images/osp-compute-0.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=107374182400 backing_file=/var/lib/libvirt/images/rhel9-guest.qcow2 backing_fmt=qcow2 lazy_refcounts=off refcount_bits=16

[root@hypervisor ~]# ls -lh /var/lib/libvirt/images/osp-compute-0.qcow2
-rw-r--r--. 1 root root 194K Aug 29 21:12 /var/lib/libvirt/images/osp-compute-0.qcow2

[root@hypervisor ~]# qemu-img info /var/lib/libvirt/images/osp-compute-0.qcow2
image: /var/lib/libvirt/images/osp-compute-0.qcow2
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 196 KiB
cluster_size: 65536
backing file: /var/lib/libvirt/images/rhel9-guest.qcow2
backing file format: qcow2
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
    extended l2: false

[root@hypervisor ~]#
```

#### 2.3.2 osp-compute0 가상머신 생성

오픈스택 컴퓨트 노드 (osp-compute0) 가상머신 인스턴스 생성
```bash
virt-install --virt-type kvm --ram 16384 --vcpus 4 --cpu=host-passthrough --os-variant rhel8.4 --disk path=/var/lib/libvirt/images/osp-compute-0.qcow2,device=disk,bus=virtio,format=qcow2 --network network:ocp4-provisioning --network network:ocp4-net --boot hd,network --noautoconsole --vnc --name osp-compute0 --noreboot
virsh dominfo osp-compute0
```
* 인스턴스만 생성하고, 상태는 *shut off* 임

출력 결과
```
[root@hypervisor ~]# virt-install --virt-type kvm --ram 16384 --vcpus 4 --cpu=host-passthrough --os-variant rhel8.4 --disk path=/var/lib/libvirt/images/osp-compute-0.qcow2,device=disk,bus=virtio,format=qcow2 --network network:ocp4-provisioning --network network:ocp4-net --boot hd,network --noautoconsole --vnc --name osp-compute0 --noreboot

Starting install...
Domain creation completed.
You can restart your domain by running:
  virsh --connect qemu:///system start osp-compute0

[root@hypervisor ~]# virsh dominfo osp-compute0
Id:             -
Name:           osp-compute0
UUID:           ee52fe87-501f-4407-970f-1ed8df3cb02a
OS Type:        hvm
State:          shut off
CPU(s):         4
Max memory:     16777216 KiB
Used memory:    16777216 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: selinux
Security DOI:   0

[root@hypervisor ~]#
```

#### 2.3.3 osp-compute0 가상머신 시작

```bash
virsh start osp-compute0
watch virsh domifaddr osp-compute0 --source agent
```

출력 결과
```
[root@hypervisor ~]# virsh start osp-compute0
Domain 'osp-compute0' started

[root@hypervisor ~]# watch virsh domifaddr osp-compute0 --source agent

 Name       MAC address          Protocol     Address
-------------------------------------------------------------------------------
 lo         00:00:00:00:00:00    ipv4         127.0.0.1/8
 -          -                    ipv6         ::1/128
 eth0       52:54:00:df:a3:48    ipv4         172.22.0.38/24
 -          -                    ipv6         fe80::fbdd:ea10:ea33:8610/64
 eth1       52:54:00:90:1b:36    ipv4         192.168.123.61/24
 -          -                    ipv6         fe80::29bf:1abe:72af:fffc/64

Ctrl+C

[root@hypervisor ~]#
```
* osp-compute0의 프로비저닝 네트워크의 IP주소 확인


#### 2.3.4 osp-compute0 네트워크 구성

osp-compute0 가상머신의 네트워크 확인
```bash
ssh root@192.168.123.61
ip --br a s
nmcli con show
nmcli --field ipv4.method con show "Wired connection 1"
nmcli --field ipv4.method con show "Wired connection 2"
```

출력 결과
```
[root@hypervisor ~]# ssh root@192.168.123.61

...<snip>...

[root@localhost ~]# ip --br a s
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.22.0.38/24 fe80::fbdd:ea10:ea33:8610/64
eth1             UP             192.168.123.61/24 fe80::29bf:1abe:72af:fffc/64

[root@localhost ~]# nmcli con show
NAME                UUID                                  TYPE      DEVICE
Wired connection 2  bed9a15e-5866-3e56-95a3-c0d3aa671b88  ethernet  eth1
Wired connection 1  20de0342-b009-36b1-b0b3-5f1dde4dcbdd  ethernet  eth0
lo                  fb433253-b3a5-4997-80fb-667255519bae  loopback  lo

[root@localhost ~]# nmcli --field ipv4.method con show "Wired connection 1"
ipv4.method:                            auto

[root@localhost ~]# nmcli --field ipv4.method con show "Wired connection 2"
ipv4.method:                            auto

[root@localhost ~]#
```

osp-compute0 가상머신의 eth0 (ocp4-provisioning) 네트워크 설정
```bash
nmcli co delete 'Wired connection 1'
nmcli con add con-name "static-eth0" ifname eth0 type ethernet ip4 172.22.0.100/24 ipv4.dns "172.22.0.89"
nmcli con up "static-eth0"
nmcli --field ipv4.method,ipv4.dns,ipv4.addresses con show static-eth0
```

출력 결과
```
[root@localhost ~]# nmcli co delete 'Wired connection 1'
Connection 'Wired connection 1' (20de0342-b009-36b1-b0b3-5f1dde4dcbdd) successfully deleted.

[root@localhost ~]# nmcli con add con-name "static-eth0" ifname eth0 type ethernet ip4 172.22.0.100/24 ipv4.dns "172.22.0.89"
Connection 'static-eth0' (a1341966-2fbf-4dc8-87bd-8d3094da000a) successfully added.

[root@localhost ~]# nmcli con up "static-eth0"
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)

[root@localhost ~]# nmcli --field ipv4.method,ipv4.dns,ipv4.addresses con show static-eth0
ipv4.method:                            manual
ipv4.dns:                               172.22.0.89
ipv4.addresses:                         172.22.0.100/24

[root@localhost ~]#
```

osp-compute0 가상머신의 eth1 (ocp4-net) 네트워크 설정
```bash
nmcli co delete 'Wired connection 2'
nmcli con add con-name "static-eth1" ifname eth1 type ethernet ip4 192.168.123.61/24 ipv4.dns "192.168.123.100" ipv4.gateway "192.168.123.1"
nmcli con up "static-eth1"
nmcli --field ipv4.method,ipv4.dns,ipv4.addresses con show static-eth1
```

출력 결과
```
[root@localhost ~]# nmcli co delete 'Wired connection 2'
Connection 'Wired connection 2' (bed9a15e-5866-3e56-95a3-c0d3aa671b88) successfully deleted.

[root@localhost ~]# nmcli con add con-name "static-eth1" ifname eth1 type ethernet ip4 192.168.123.61/24 ipv4.dns "192.168.123.100" ipv4.gateway "192.168.123.1"

[root@localhost ~]# nmcli con up "static-eth1"
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)

[root@localhost ~]# nmcli --field ipv4.method,ipv4.dns,ipv4.addresses con show static-eth1
ipv4.method:                            manual
ipv4.dns:                               192.168.123.100
ipv4.addresses:                         192.168.123.61/24

[root@localhost ~]#
```

#### 2.3.5 호스트 이름 설정

시작된 osp-compute0 가상머신 인스턴스의 호스트 이름 설정
```bash
hostnamectl set-hostname edpm-compute-0.aio.example.com
hostnamectl status
```

출력 결과
```
[root@localhost ~]# hostnamectl set-hostname edpm-compute-0.aio.example.com

[root@localhost ~]# hostnamectl status
 Static hostname: edpm-compute-0.aio.example.com
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 19438eecfc8bc46cd485c1fe44a822af
         Boot ID: 50c36a8623924a0aa1585be37d7f3b30
  Virtualization: kvm
Operating System: Red Hat Enterprise Linux 9.4 (Plow)
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-427.13.1.el9_4.x86_64
    Architecture: x86-64
 Hardware Vendor: Red Hat
  Hardware Model: KVM
Firmware Version: 1.16.0-4.module+el8.9.0+19570+14a90618

[root@localhost ~]#
```
<br>

### 2.4 osp-compute0 리포지토리 구성

#### 2.4.1 필요한 인증서 설정

```bash
curl -ko /etc/pki/ca-trust/source/anchors/demosat-ha.infra.demo.redhat.com.ca.crt  "https://demosat-ha.infra.demo.redhat.com/pub/katello-server-ca.crt"
update-ca-trust
```

출력 결과
```
[root@edpm-compute-0 ~]# curl -ko /etc/pki/ca-trust/source/anchors/demosat-ha.infra.demo.redhat.com.ca.crt  "https://demosat-ha.infra.demo.redhat.com/pub/katello-server-ca.crt"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2529  100  2529    0     0   7184      0 --:--:-- --:--:-- --:--:--  7184

[root@edpm-compute-0 ~]# update-ca-trust

[root@edpm-compute-0 ~]#
```

#### 2.4.2 서브스크립션 관련 패키지 설치 및 등록

```bash
yum install -y "https://demosat-ha.infra.demo.redhat.com/pub/katello-ca-consumer-latest.noarch.rpm"
subscription-manager register --org="Red_Hat_RHDP_Labs"  --activationkey="demosat-smt-b1374-1711111157" --serverurl=https://demosat-ha.infra.demo.redhat.com:8443/rhsm --baseurl=https://demosat-ha.infra.demo.redhat.com/pulp/repos
```

출력 결과
```
[root@edpm-compute-0 ~]# yum install -y "https://demosat-ha.infra.demo.redhat.com/pub/katello-ca-consumer-latest.noarch.rpm"

...<snip>...

[root@edpm-compute-0 ~]# subscription-manager register --org="Red_Hat_RHDP_Labs"  --activationkey="demosat-smt-b1374-1711111157" --serverurl=https://demosat-ha.infra.demo.redhat.com:8443/rhsm --baseurl=https://demosat-ha.infra.demo.redhat.com/pulp/repos
The system has been registered with ID: 8c117321-d7fe-4ba6-91bf-4c4b340c1c7b
The registered system name is: edpm-compute-0.aio.example.com

[root@edpm-compute-0 ~]#
```

#### 2.4.3 필요한 리포지토리 설정

```bash
subscription-manager repos --disable=*
subscription-manager repos --enable=rhceph-6-tools-for-rhel-9-x86_64-rpms --enable=rhel-9-for-x86_64-baseos-rpms --enable=rhel-9-for-x86_64-appstream-rpms --enable=rhel-9-for-x86_64-highavailability-rpms --enable=openstack-dev-preview-for-rhel-9-x86_64-rpms --enable=fast-datapath-for-rhel-9-x86_64-rpms
```
* 오픈스택 컴퓨트 노드 상에 Ceph 연동을 위한 패키지 설치를 위해 관련 리포지토리 설정

출력 결과
```
[root@edpm-compute-0 ~]# subscription-manager repos --disable=*

[root@edpm-compute-0 ~]# subscription-manager repos --enable=rhceph-6-tools-for-rhel-9-x86_64-rpms --enable=rhel-9-for-x86_64-baseos-rpms --enable=rhel-9-for-x86_64-appstream-rpms --enable=rhel-9-for-x86_64-highavailability-rpms --enable=openstack-dev-preview-for-rhel-9-x86_64-rpms --enable=fast-datapath-for-rhel-9-x86_64-rpms
Repository 'openstack-17.1-for-rhel-9-x86_64-rpms' is disabled for this system.
Repository 'fast-datapath-for-rhel-9-x86_64-rpms' is disabled for this system.
Repository 'rhel-9-for-x86_64-highavailability-rpms' is disabled for this system.
Repository 'rhel-9-for-x86_64-baseos-rpms' is disabled for this system.
Repository 'rhel-9-for-x86_64-appstream-rpms' is disabled for this system.
Repository 'openstack-dev-preview-for-rhel-9-x86_64-rpms' is disabled for this system.
Repository 'rhceph-6-tools-for-rhel-9-x86_64-rpms' is disabled for this system.
Repository 'rhceph-6-tools-for-rhel-9-x86_64-rpms' is enabled for this system.
Repository 'rhel-9-for-x86_64-baseos-rpms' is enabled for this system.
Repository 'rhel-9-for-x86_64-appstream-rpms' is enabled for this system.
Repository 'rhel-9-for-x86_64-highavailability-rpms' is enabled for this system.
Repository 'openstack-dev-preview-for-rhel-9-x86_64-rpms' is enabled for this system.
Repository 'fast-datapath-for-rhel-9-x86_64-rpms' is enabled for this system.

[root@edpm-compute-0 ~]#
```
<br>

### 2.5 필요한 패키지 설치

```bash
yum install -y "https://demosat-ha.infra.demo.redhat.com/pub/katello-ca-consumer-latest.noarch.rpm"
```

출력 결과
```
[root@edpm-compute-0 ~]# yum install -y "https://demosat-ha.infra.demo.redhat.com/pub/katello-ca-consumer-latest.noarch.rpm"

...<snip>...

[root@edpm-compute-0 ~]#
```
<br>

### 2.6 osp-compute0 가상머신 스냅샷 생성

```bash
virsh snapshot-create-as osp-compute0 preprovisioned
virsh snapshot-list osp-compute0
virsh snapshot-info osp-compute0 --current
```

출력 결과
```
[root@hypervisor ~]# virsh snapshot-create-as osp-compute0 preprovisioned
Domain snapshot preprovisioned created

[root@hypervisor ~]# virsh snapshot-list osp-compute0
 Name             Creation Time               State
-------------------------------------------------------
 preprovisioned   2024-08-30 00:27:08 -0400   running

[root@hypervisor ~]# virsh snapshot-info osp-compute0 --current
Name:           preprovisioned
Domain:         osp-compute0
Current:        yes
State:          running
Location:       internal
Parent:         -
Children:       0
Descendants:    0
Metadata:       yes

[root@hypervisor ~]#
```
<br>

### 2.7 osp-compute0에 SSH 키 복사

가상머신 이미지 구성 시에 추가 SSH 키를 베이스천에 복사
```bash
scp /root/.ssh/id_rsa root@192.168.123.100:/root/.ssh/id_rsa_compute
scp /root/.ssh/id_rsa.pub root@192.168.123.100:/root/.ssh/id_rsa_compute.pub
ssh root@192.168.123.100 "ls -lh /root/.ssh/id_rsa_compute*"
```

출력 결과
```
[root@hypervisor ~]# scp /root/.ssh/id_rsa root@192.168.123.100:/root/.ssh/id_rsa_compute
id_rsa                                                                 100% 3357     4.9MB/s   00:00

[root@hypervisor ~]# scp /root/.ssh/id_rsa.pub root@192.168.123.100:/root/.ssh/id_rsa_compute.pub
id_rsa.pub                                                             100%  726     1.1MB/s   00:00

[root@hypervisor ~]# ssh root@192.168.123.100 "ls -lh /root/.ssh/id_rsa_compute*"
-rw-------. 1 root root 3.3K Aug 30 00:40 /root/.ssh/id_rsa_compute
-rw-r--r--. 1 root root  726 Aug 30 00:40 /root/.ssh/id_rsa_compute.pub

[root@hypervisor ~]#
```
<br>

## 3. 데이터-플레인 설정 끝내기

### 3.1 시크릿 생성

#### 3.1.1 앤서블 SSH 프라이빗 시크릿 생성

```bash
#oc create secret generic dataplane-ansible-ssh-private-key-secret --save-config --dry-run=client --from-file=authorized_keys=/root/.ssh/id_rsa_compute.pub --from-file=ssh-privatekey=/root/.ssh/id_rsa_compute --from-file=ssh-publickey=/root/.ssh/id_rsa_compute.pub -n openstack -o yaml > dataplane-ansible-ssh-private-key-secret.yaml
#oc apply -f dataplane-ansible-ssh-private-key-secret.yaml
oc create secret generic dataplane-ansible-ssh-private-key-secret --save-config --dry-run=client --from-file=authorized_keys=/root/.ssh/id_rsa_compute.pub --from-file=ssh-privatekey=/root/.ssh/id_rsa_compute --from-file=ssh-publickey=/root/.ssh/id_rsa_compute.pub -n openstack -o yaml | oc apply -f-
oc get secret/dataplane-ansible-ssh-private-key-secret -n openstack
```

앤서블 SSH 프라이빗 키 시크릿 예
```yaml
apiVersion: v1
data:
  authorized_keys: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVB...<snip>...NLaXc9PSAK
  ssh-privatekey: LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBV...<snip>...IIFBSSVZBVEUgS0VZLS0tLS0K
  ssh-publickey: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQU...<snip>...wVEJjald2cUZRVUNLaXc9PSAK
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"kind":"Secret","apiVersion":"v1","metadata":{"name":"dataplane-ansible-ssh-private-key-secret","namespace":"openstack","creationTimestamp":null},"data":{"authorized_keys":"c3NoLXJzYSBBQUFBQjNOemFDMXljMkVB...<snip>...NLaXc9PSAK","ssh-privatekey":"LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBV...<snip>...IIFBSSVZBVEUgS0VZLS0tLS0K","ssh-publickey":"c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQU...<snip>...wVEJjald2cUZRVUNLaXc9PSAK"}}
  creationTimestamp: null
  name: dataplane-ansible-ssh-private-key-secret
  namespace: openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc create secret generic dataplane-ansible-ssh-private-key-secret --save-config --dry-run=client --from-file=authorized_keys=/root/.ssh/id_rsa_compute.pub --from-file=ssh-privatekey=/root/.ssh/id_rsa_compute --from-file=ssh-publickey=/root/.ssh/id_rsa_compute.pub -n openstack -o yaml > dataplane-ansible-ssh-private-key-secret.yaml

[root@ocp4-bastion ~]# oc apply -f dataplane-ansible-ssh-private-key-secret.yaml
secret/dataplane-ansible-ssh-private-key-secret created

[root@ocp4-bastion ~]# oc get secret/dataplane-ansible-ssh-private-key-secret -n openstack
NAME                                       TYPE     DATA   AGE
dataplane-ansible-ssh-private-key-secret   Opaque   3      2m2s

[root@ocp4-bastion ~]#
```

#### 3.1.2 Nova 마이그레이션 SSH 키 시크릿 생성

```bash
ssh-keygen -f ./id -t ecdsa-sha2-nistp521 -N ''
cat ./id
#oc create secret generic nova-migration-ssh-key --from-file=ssh-privatekey=id --from-file=ssh-publickey=id.pub -n openstack -o yaml > nova-migration-ssh-key.yaml
#oc apply -f nova-migration-ssh-key.yaml
oc create secret generic nova-migration-ssh-key --from-file=ssh-privatekey=id --from-file=ssh-publickey=id.pub -n openstack -o yaml | oc apply -f-
oc get secret/nova-migration-ssh-key -n openstack
```

출력 결과
```
[root@ocp4-bastion ~]# ssh-keygen -f ./id -t ecdsa-sha2-nistp521 -N ''
Generating public/private ecdsa-sha2-nistp521 key pair.
Your identification has been saved in ./id
Your public key has been saved in ./id.pub
The key fingerprint is:
SHA256:Eu/vTkvo+dUP/yH8iPxBMzWSIel0B2zHmanN1j1bRQU root@ocp4-bastion.aio.example.com
The key's randomart image is:
+---[ECDSA 521]---+
|           .ooEoB|
|           o.+oB.|
|      .   o oo*o+|
|       o   . .o==|
|      . S    +. +|
|       o .  o.o. |
|        o o .+o. |
|       . =.o. =+.|
|        o+*o.o .=|
+----[SHA256]-----+

[root@ocp4-bastion ~]# cat id.pub
ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAFRFpB7jmxK+x6t1lelmmiSdn3ShYeIxFcVelTpZp2ZYgDdAaXGVcqCEaI5MobgtbWgsZ56ZAb+a+/yHScZafnhXACTIvK93guPSo0Q9PL09q4uWCqL6J0pyv3yLa+9i3JJubUneN99xC8TCHtQ0wY0j2BfjoTnhNH6dCSwIsCbumutQA== root@ocp4-bastion.aio.example.com

[root@ocp4-bastion ~]# oc create secret generic nova-migration-ssh-key --from-file=ssh-privatekey=id --from-file=ssh-publickey=id.pub -n openstack -o yaml > nova-migration-ssh-key.yaml

[root@ocp4-bastion ~]# oc apply -f nova-migration-ssh-key.yaml
Warning: resource secrets/nova-migration-ssh-key is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by oc apply. oc apply should only be used on resources created declaratively by either oc create --save-config or oc apply. The missing annotation will be patched automatically.
secret/nova-migration-ssh-key configured

[root@ocp4-bastion ~]# oc get secret/nova-migration-ssh-key -n openstack
NAME                     TYPE     DATA   AGE
nova-migration-ssh-key   Opaque   2      73s

[root@ocp4-bastion ~]#
```
<br>

### 3.2 데이터-플레인 배포

#### 3.2.1 데이터 플레인 노드 설정

노드 설정 YAML 파일
```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneNodeSet
metadata:
  name: openstack-edpm-ipam
spec:
  tlsEnabled: true
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
  preProvisioned: true
  networkAttachments:
    - ctlplane
  nodes:
    edpm-compute-0:
      hostName: edpm-compute-0
      ansible:
        ansibleHost: 172.22.0.100
      networks:
        - name: ctlplane
          subnetName: subnet1
          defaultRoute: false
          fixedIP: 172.22.0.100
        - name: internalapi
          subnetName: subnet1
        - name: storage
          subnetName: subnet1
        - name: tenant
          subnetName: subnet1
        - name: external
          subnetName: subnet1
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

노드 설정
```bash
oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-node-set-deploy.yaml
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-node-set-deploy.yaml
openstackdataplanenodeset.dataplane.openstack.org/openstack-edpm-ipam created

[root@ocp4-bastion ~]# oc get openstackdataplanenodeset
NAME                  STATUS   MESSAGE
openstack-edpm-ipam   False    Deployment not started

[root@ocp4-bastion ~]#
```
* *MESSAGE*가 *Deployment not started*인 것을 확인

#### 3.2.2 데이터-플레인 노드 배포

배포 YAML 파일
```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneDeployment
metadata:
  name: openstack-edpm-ipam
spec:
  nodeSets:
    - openstack-edpm-ipam
```

배포 실행
```bash
oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-deployment.yaml
oc logs -l app=openstackansibleee -f --max-log-requests 10
oc get openstackdataplanedeployment
oc get openstackdataplanenodeset
```

출력 결과
```
[root@ocp4-bastion ~]# oc apply -f $HOME/labrepo/content/files/osp-ng-dataplane-deployment.yaml
openstackdataplanedeployment.dataplane.openstack.org/openstack-edpm-ipam created

[root@ocp4-bastion ~]# oc logs -l app=openstackansibleee -f --max-log-requests 10

...<snip>...

PLAY RECAP *********************************************************************
edpm-compute-0             : ok=55   changed=27   unreachable=0    failed=0    skipped=63   rescued=2    ignored=0

[root@ocp4-bastion ~]# oc get openstackdataplanedeployment
NAME                  NODESETS                  STATUS   MESSAGE
openstack-edpm-ipam   ["openstack-edpm-ipam"]   False    Deployment in progress

...

[root@ocp4-bastion ~]# oc get openstackdataplanedeployment
NAME                  NODESETS                  STATUS   MESSAGE
openstack-edpm-ipam   ["openstack-edpm-ipam"]   True     Setup complete

[root@ocp4-bastion ~]# oc get openstackdataplanenodeset
NAME                  STATUS   MESSAGE
openstack-edpm-ipam   True     NodeSet Ready

[root@ocp4-bastion ~]#
```
* 앤서블 로그 확인 시 기본 값이 5이므로, --max-log-requests를 10, 20 등으로 바꾸어서 확인
* 앤서블 로그가 실행되는 것을 확인
* *MESSAGE*가 *Deployment in progress*에서 *Setup Complete*가 될 때까지 기다림
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 컨트롤-플레인 생성 <<](./create-oso-ctl-plane.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 접속 >>](./access-openstack.md)