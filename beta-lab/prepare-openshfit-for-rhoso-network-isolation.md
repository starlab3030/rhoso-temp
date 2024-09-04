# 오픈스택 서비스의 네트워크 격리를 위해 오픈시프트 준비

목차
1. [네트워크 구성 파일 확인](./prepare-openshfit-for-rhoso-network-isolation.md#1-네트워크-구성-파일-확인)<br>
2. [워커 노드 네트워크 구성](./prepare-openshfit-for-rhoso-network-isolation.md#2-워커-노드-네트워크-구성)<br>
   2.1 [오픈스택 서비스 네트워크 구성](./prepare-openshfit-for-rhoso-network-isolation.md#21-오픈스택-서비스-네트워크-구성)<br>
   2.2 [첫 번째 워커 노드의 네트워크 구성](./prepare-openshfit-for-rhoso-network-isolation.md#22-첫-번째-워커-노드의-네트워크-구성)<br>
   2.3 [두 번째 워커 노드의 네트워크 구성](./prepare-openshfit-for-rhoso-network-isolation.md#23-두-번째-워커-노드의-네트워크-구성)<br>
   2.4 [세 번째 워커 노드의 네트워크 구성](./prepare-openshfit-for-rhoso-network-isolation.md#24-세-번째-워커-노드의-네트워크-구성)<br>
3. [NAD 구성](./prepare-openshfit-for-rhoso-network-isolation.md#3-networkattachmentdefinition-nad-구성)<br>
4. [MetalLB 구성](./prepare-openshfit-for-rhoso-network-isolation.md#4-metallb-구성)<br>
5. [요약](./prepare-openshfit-for-rhoso-network-isolation.md#요약)<br>
<br>

## 1. 네트워크 구성 파일 확인

오픈시프트 워커 노드 3대의 네트워크 구성이 필요합니다.

네트워크 구성 파일 확인
```bash
cd $HOME/labrepo/content/file
ls -lh osp-ng-nncp-*.yaml
```

출력 결과
```
[root@ocp4-bastion files]# cd $HOME/labrepo/content/file

[root@ocp4-bastion files]# ls -lh osp-ng-nncp-*.yaml
-rw-r--r--. 1 root root 1.5K Aug 28 23:39 osp-ng-nncp-w1.yaml
-rw-r--r--. 1 root root 1.5K Aug 28 23:39 osp-ng-nncp-w2.yaml
-rw-r--r--. 1 root root 1.5K Aug 28 23:39 osp-ng-nncp-w3.yaml

[root@ocp4-bastion files]# 
```
<br>

## 2. 워커 노드 네트워크 구성

## 2.1 오픈스택 서비스 네트워크 구성

네트워크 레이아웃
|$\color{lime}{\texttt{용도}}$|$\color{lime}{\texttt{IP 대역}}$|$\color{lime}{\texttt{타입}}$|
|:---|:---:|---:|
|`Internal API`|172.17.0.0/24|vlan 20|
|`Storage`|172.18.0.0/24|vlan 21|
|`tenant`|172.19.0.0/24|vlan 22|
|`base`|172.22.0.0/24|ethernet|
<br>

### 2.2 첫 번째 워커 노드의 네트워크 구성

ocp4-worker1 노드의 NNCP 구성을 위한 YAML 파일
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker1
spec:
  desiredState:
    interfaces:
    - description: internalapi vlan interface
      ipv4:
        address:
        - ip: 172.17.0.10
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.20
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 20
    - description: storage vlan interface
      ipv4:
        address:
        - ip: 172.18.0.10
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.21
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 21
    - description: tenant vlan interface
      ipv4:
        address:
        - ip: 172.19.0.10
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.22
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 22
    - description: Configuring enp1s0
      ipv4:
        address:
        - ip: 172.22.0.10
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      mtu: 1500
      name: enp1s0
      state: up
      type: ethernet
  nodeSelector:
    kubernetes.io/hostname: ocp4-worker1.aio.example.com
    node-role.kubernetes.io/worker: ""
```

ocp4-worker1 노드의 NNCP 구성
```bash
ssh core@192.168.123.104 "ip --br a"
oc apply -f osp-ng-nncp-w1.yaml 
oc get nncp
ssh core@192.168.123.104 "ip --br a"
```

출력 결과
```
[root@ocp4-bastion files]# ssh core@192.168.123.104 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.68/24 fe80::dcad:beff:feef:4/64 
enp2s0           UP             
enp3s0           UP             192.168.3.108/24 fe80::5054:ff:fe00:104/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.104/24 169.254.169.2/29 fe80::5054:ff:fe00:4/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.129.2.2/23 fe80::d442:85ff:fe20:7c3a/64 
genev_sys_6081   UNKNOWN        fe80::189a:7ff:feb4:d48d/64 

...<snip>...

[root@ocp4-bastion files]# oc apply -f osp-ng-nncp-w1.yaml 
nodenetworkconfigurationpolicy.nmstate.io/osp-enp1s0-worker-ocp4-worker1 created

[root@ocp4-bastion files]# oc get nncp
NAME                             STATUS      REASON
osp-enp1s0-worker-ocp4-worker1   Available   SuccessfullyConfigured

[root@ocp4-bastion files]# ssh core@192.168.123.104 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.10/24 
enp2s0           UP             
enp3s0           UP             192.168.3.108/24 fe80::5054:ff:fe00:104/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.104/24 169.254.169.2/29 fe80::5054:ff:fe00:4/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.129.2.2/23 fe80::d442:85ff:fe20:7c3a/64 
genev_sys_6081   UNKNOWN        fe80::189a:7ff:feb4:d48d/64 

...<snip>...

enp1s0.20@enp1s0 UP             172.17.0.10/24 
enp1s0.21@enp1s0 UP             172.18.0.10/24 
enp1s0.22@enp1s0 UP             172.19.0.10/24 

[root@ocp4-bastion files]#
```
* vlan 기반으로 추가된 네트워크 대역 확인
<br>

### 2.3 두 번째 워커 노드의 네트워크 구성

ocp4-worker2 노드의 NNCP 구성을 위한 YAML 파일
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker2
spec:
  desiredState:
    interfaces:
    - description: internalapi vlan interface
      ipv4:
        address:
        - ip: 172.17.0.11
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.20
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 20
    - description: storage vlan interface
      ipv4:
        address:
        - ip: 172.18.0.11
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.21
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 21
    - description: tenant vlan interface
      ipv4:
        address:
        - ip: 172.19.0.11
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.22
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 22
    - description: Configuring enp1s0
      ipv4:
        address:
        - ip: 172.22.0.11
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      mtu: 1500
      name: enp1s0
      state: up
      type: ethernet
  nodeSelector:
    kubernetes.io/hostname: ocp4-worker2.aio.example.com 
    node-role.kubernetes.io/worker: ""
```

ocp4-worker2 노드의 NNCP 구성
```bash
ssh core@192.168.123.105 "ip --br a"
```

출력 결과
```
[root@ocp4-bastion files]# ssh core@192.168.123.105 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.69/24 fe80::dcad:beff:feef:5/64 
enp2s0           UP             
enp3s0           UP             192.168.3.109/24 fe80::5054:ff:fe00:105/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.105/24 169.254.169.2/29 fe80::5054:ff:fe00:5/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.128.2.2/23 fe80::9c53:abff:fe94:2bae/64 
genev_sys_6081   UNKNOWN        fe80::c4cc:12ff:fedd:37b7/64 

...<snip>...

[root@ocp4-bastion files]# oc apply -f osp-ng-nncp-w2.yaml 
nodenetworkconfigurationpolicy.nmstate.io/osp-enp1s0-worker-ocp4-worker2 created

[root@ocp4-bastion files]# oc get nncp -w
NAME                             STATUS      REASON
osp-enp1s0-worker-ocp4-worker1   Available   SuccessfullyConfigured
osp-enp1s0-worker-ocp4-worker2   Available   SuccessfullyConfigured

[root@ocp4-bastion files]# ssh core@192.168.123.105 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.11/24 
enp2s0           UP             
enp3s0           UP             192.168.3.109/24 fe80::5054:ff:fe00:105/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.105/24 169.254.169.2/29 fe80::5054:ff:fe00:5/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.128.2.2/23 fe80::9c53:abff:fe94:2bae/64 
genev_sys_6081   UNKNOWN        fe80::c4cc:12ff:fedd:37b7/64 

...<snip>...

enp1s0.20@enp1s0 UP             172.17.0.11/24 
enp1s0.21@enp1s0 UP             172.18.0.11/24 
enp1s0.22@enp1s0 UP             172.19.0.11/24 

[root@ocp4-bastion files]# 
```
* vlan으로 추가된 네트워크 확인
<br>

### 2.4 세 번째 워커 노드의 네트워크 구성

ocp4-worker3 노드의 NNCP 구성을 위한 YAML 파일
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker3
spec:
  desiredState:
    interfaces:
    - description: internalapi vlan interface
      ipv4:
        address:
        - ip: 172.17.0.12
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.20
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 20
    - description: storage vlan interface
      ipv4:
        address:
        - ip: 172.18.0.12
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.21
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 21
    - description: tenant vlan interface
      ipv4:
        address:
        - ip: 172.19.0.12
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      name: enp1s0.22
      state: up
      type: vlan
      vlan:
        base-iface: enp1s0
        id: 22
    - description: Configuring enp1s0
      ipv4:
        address:
        - ip: 172.22.0.12
          prefix-length: 24 
        enabled: true
        dhcp: false
      ipv6:
        enabled: false
      mtu: 1500
      name: enp1s0
      state: up
      type: ethernet
  nodeSelector:
    kubernetes.io/hostname: ocp4-worker3.aio.example.com 
    node-role.kubernetes.io/worker: ""
```

ocp4-worker3 노드의 NNCP 구성
```bash
ssh core@192.168.123.106 "ip --br a"

ssh core@192.168.123.106 "ip --br a"
```

출력 결과
```
[root@ocp4-bastion files]# ssh core@192.168.123.106 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.70/24 fe80::dcad:beff:feef:6/64 
enp2s0           UP             
enp3s0           UP             192.168.3.110/24 fe80::5054:ff:fe00:106/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.106/24 169.254.169.2/29 192.168.123.11/32 fe80::5054:ff:fe00:6/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.131.0.2/23 fe80::48f2:8fff:fed3:32ad/64 
genev_sys_6081   UNKNOWN        fe80::d8bd:ecff:fe84:f270/64 

...<snip>...

[root@ocp4-bastion files]# oc apply -f osp-ng-nncp-w3.yaml 
nodenetworkconfigurationpolicy.nmstate.io/osp-enp1s0-worker-ocp4-worker3 created

[root@ocp4-bastion files]# oc get nncp -w
NAME                             STATUS      REASON
osp-enp1s0-worker-ocp4-worker1   Available   SuccessfullyConfigured
osp-enp1s0-worker-ocp4-worker2   Available   SuccessfullyConfigured
osp-enp1s0-worker-ocp4-worker3   Available   SuccessfullyConfigured

...<snip>...

[root@ocp4-bastion files]# ssh core@192.168.123.106 "ip --br a"
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.22.0.12/24 
enp2s0           UP             
enp3s0           UP             192.168.3.110/24 fe80::5054:ff:fe00:106/64 
ovs-system       DOWN           
br-ex            UNKNOWN        192.168.123.106/24 169.254.169.2/29 192.168.123.11/32 fe80::5054:ff:fe00:6/64 
br-int           DOWN           
ovn-k8s-mp0      UNKNOWN        10.131.0.2/23 fe80::48f2:8fff:fed3:32ad/64 
genev_sys_6081   UNKNOWN        fe80::d8bd:ecff:fe84:f270/64 

...<snip>...

enp1s0.20@enp1s0 UP             172.17.0.12/24 
enp1s0.21@enp1s0 UP             172.18.0.12/24 
enp1s0.22@enp1s0 UP             172.19.0.12/24 

[root@ocp4-bastion files]#
```
* vlan으로 추가된 네트워크 확인
<br>

## 3. NetworkAttachmentDefinition (NAD) 구성

각각의 격리된 네트워크를 서비스 포드의 네트워크에 추가하기 위해 NAD 리소스 구성이 필요합니다.

### 3.1 NAD 구성 파일 확인

NAD 파일 내용 확인
```bash
yq -y '.' osp-ng-netattach.yaml
```

NAD 구성을 위한 YAML 파일
```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: ctlplane
  namespace: openstack
spec:
  config: "{\n  \"cniVersion\": \"0.3.1\",\n  \"name\": \"ctlplane\",\n  \"type\"\
    : \"macvlan\",\n  \"master\": \"enp1s0\",\n  \"ipam\": {\n    \"type\": \"whereabouts\"\
    ,\n    \"range\": \"172.22.0.0/24\",\n    \"range_start\": \"172.22.0.30\",\n\
    \    \"range_end\": \"172.22.0.70\"\n  }\n}\n"
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: internalapi
  namespace: openstack
spec:
  config: "{\n  \"cniVersion\": \"0.3.1\",\n  \"name\": \"internalapi\",\n  \"type\"\
    : \"macvlan\",\n  \"master\": \"enp1s0.20\", \n  \"ipam\": { \n    \"type\": \"\
    whereabouts\", \n    \"range\": \"172.17.0.0/24\",\n    \"range_start\": \"172.17.0.30\"\
    , \n    \"range_end\": \"172.17.0.70\"\n  }\n}\n"
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: storage
  namespace: openstack
spec:
  config: "{\n  \"cniVersion\": \"0.3.1\",\n  \"name\": \"storage\",\n  \"type\":\
    \ \"macvlan\",\n  \"master\": \"enp1s0.21\", \n  \"ipam\": {\n    \"type\": \"\
    whereabouts\",\n    \"range\": \"172.18.0.0/24\",\n    \"range_start\": \"172.18.0.30\"\
    ,\n    \"range_end\": \"172.18.0.70\"\n  }\n}\n"
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: tenant
  namespace: openstack
spec:
  config: "{\n  \"cniVersion\": \"0.3.1\",\n  \"name\": \"tenant\",\n  \"type\": \"\
    macvlan\",\n  \"master\": \"enp1s0.22\",\n  \"ipam\": {\n    \"type\": \"whereabouts\"\
    ,\n    \"range\": \"172.19.0.0/24\",\n    \"range_start\": \"172.19.0.30\",\n\
    \    \"range_end\": \"172.19.0.70\"\n  }\n}\n"
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: external
  namespace: openstack
spec:
  config: "{\n  \"cniVersion\": \"0.3.1\",\n  \"name\": \"external\",\n  \"type\"\
    : \"macvlan\",\n  \"master\": \"enp3s0\",\n  \"ipam\": {\n    \"type\": \"whereabouts\"\
    ,\n    \"range\": \"192.168.123.0/24\",\n    \"range_start\": \"192.168.123.60\"\
    ,\n    \"range_end\": \"192.168.123.110\"\n    }\n  }"
```

NAD 리소스 각각의 구성 정보 확인
```bash
yq -r '.spec.config' osp-ng-netattach.yaml
```

출력 결과 JSON 형식
```json
{  
  "cniVersion": "0.3.1",
  "name": "ctlplane",
  "type": "macvlan",
  "master": "enp1s0",
  "ipam": {
    "type": "whereabouts",
    "range": "172.22.0.0/24",
    "range_start": "172.22.0.30",
    "range_end": "172.22.0.70"
  }
}

{
  "cniVersion": "0.3.1",
  "name": "internalapi",
  "type": "macvlan",
  "master": "enp1s0.20", 
  "ipam": { 
    "type": "whereabouts", 
    "range": "172.17.0.0/24",
    "range_start": "172.17.0.30", 
    "range_end": "172.17.0.70"
  }
}

{
  "cniVersion": "0.3.1",
  "name": "storage",
  "type": "macvlan",
  "master": "enp1s0.21", 
  "ipam": {
    "type": "whereabouts",
    "range": "172.18.0.0/24",
    "range_start": "172.18.0.30",
    "range_end": "172.18.0.70"
  }
}

{
  "cniVersion": "0.3.1",
  "name": "tenant",
  "type": "macvlan",
  "master": "enp1s0.22",
  "ipam": {
    "type": "whereabouts",
    "range": "172.19.0.0/24",
    "range_start": "172.19.0.30",
    "range_end": "172.19.0.70"
  }
}

{
  "cniVersion": "0.3.1",
  "name": "external",
  "type": "macvlan",
  "master": "enp3s0",
  "ipam": {
    "type": "whereabouts",
    "range": "192.168.123.0/24",
    "range_start": "192.168.123.60",
    "range_end": "192.168.123.110"
    }
  }
```
<br>

### 3.2 NAD 적용

```bash
oc apply -f osp-ng-netattach.yaml
```

출력 결과
```
[root@ocp4-bastion files]# oc apply -f osp-ng-netattach.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/ctlplane created
networkattachmentdefinition.k8s.cni.cncf.io/internalapi created
networkattachmentdefinition.k8s.cni.cncf.io/storage created
networkattachmentdefinition.k8s.cni.cncf.io/tenant created
networkattachmentdefinition.k8s.cni.cncf.io/external created

[root@ocp4-bastion files]#
```
<br>

## 4. MetalLB 구성

### 4.1 MetalLB의 IP 주소 범위 지정

```bash
yq -y '.' osp-ng-metal-lb-ip-address-pools.yaml
```

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: ctlplane
spec:
  addresses:
    - 172.22.0.80-172.22.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: internalapi
spec:
  addresses:
    - 172.17.0.80-172.17.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: storage
spec:
  addresses:
    - 172.18.0.80-172.18.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: tenant
spec:
  addresses:
    - 172.19.0.80-172.19.0.90
```

출력 결과
```
[root@ocp4-bastion files]# oc apply -f osp-ng-metal-lb-ip-address-pools.yaml
ipaddresspool.metallb.io/ctlplane created
ipaddresspool.metallb.io/internalapi created
ipaddresspool.metallb.io/storage created
ipaddresspool.metallb.io/tenant created

[root@ocp4-bastion files]#
```
<br>

### 4.2 L2Advertisement 리소스 구성

```bash
yq -y '.' osp-ng-metal-lb-l2-advertisements.yaml
```

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: ctlplane
  namespace: metallb-system
spec:
  ipAddressPools:
    - ctlplane
  interfaces:
    - enp1s0
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: internalapi
  namespace: metallb-system
spec:
  ipAddressPools:
    - internalapi
  interfaces:
    - enp1s0.20
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: storage
  namespace: metallb-system
spec:
  ipAddressPools:
    - storage
  interfaces:
    - enp1s0.21
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: tenant
  namespace: metallb-system
spec:
  ipAddressPools:
    - tenant
  interfaces:
    - enp1s0.22
```

출력 결과
```
[root@ocp4-bastion files]# oc apply -f osp-ng-metal-lb-l2-advertisements.yaml
l2advertisement.metallb.io/ctlplane created
l2advertisement.metallb.io/internalapi created
l2advertisement.metallb.io/storage created
l2advertisement.metallb.io/tenant created

[root@ocp4-bastion files]# 
```
<br>

### 4.3 네트워크 백엔드 확인 및 구성

오픈시프트 4.14 이상이고, OVNKubernetes를 네트워크 백엔드로 사용한다면, MetalLB가 두 번 째 네트워크 인터페이스로 동작할 수 있도록 글로벌 포워딩을 활성화 해야 합니다.

네트워크 백엔드 확인
```bash
oc get network.operator cluster --output=jsonpath='{.spec.defaultNetwork.type}'
```

출력 결과
```
[root@ocp4-bastion files]# oc get network.operator cluster --output=jsonpath='{.spec.defaultNetwork.type}'
OVNKubernetes

[root@ocp4-bastion files]#
```

네트워크 백엔드가 OVNKubernetes인 경우, IP 글로벌 포워딩 활성화
```bash
oc patch network.operator cluster -p '{"spec":{"defaultNetwork":{"ovnKubernetesConfig":{"gatewayConfig":{"ipForwarding": "Global"}}}}}' --type=merge
```

출력 결과
```
[root@ocp4-bastion files]# oc patch network.operator cluster -p '{"spec":{"defaultNetwork":{"ovnKubernetesConfig":{"gatewayConfig":{"ipForwarding": "Global"}}}}}' --type=merge
network.operator.openshift.io/cluster patched

[root@ocp4-bastion files]#
```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 보안 접근 구성 <<](./provide-secure-access-to-rhoso.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 컨트롤-플레인 생성 >>](./create-oso-ctl-plane.md)