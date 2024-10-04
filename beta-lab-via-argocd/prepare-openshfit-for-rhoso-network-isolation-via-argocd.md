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

## 1. 오픈스택 네임스페이스 생성

~/openstack-namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```
<br>

## 2. NNCP를 사용하여 노드 네트워크 구성

### 2.1 첫 번째 워커 노드의 네트워크 구성

~/osp-ng-nncp-w1.yaml
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker1
  annotations:
    argocd.argoproj.io/sync-wave: "0"
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

### 2.2 첫 번째 워커 노드의 네트워크 구성

~/osp-ng-nncp-w2.yaml
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker2
  annotations:
    argocd.argoproj.io/sync-wave: "0"
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

### 2.3 첫 번째 워커 노드의 네트워크 구성

~/osp-ng-nncp-w3.yaml
```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: osp-enp1s0-worker-ocp4-worker3
  annotations:
    argocd.argoproj.io/sync-wave: "0"
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
<br>

## 3. Network Attachment Definition (NAD) 구성

~/osp-ng-netattach.yaml
```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: ctlplane
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  config: |
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
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: internalapi
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  config: |
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
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: storage
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  config: |
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
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: tenant
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  config: |
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
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: external
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  config: |
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

## 4. MetalLB 구성

### 4.1 MetalLB의 IP 주소 범위 지정

~/osp-ng-metal-lb-ip-address-pools.yaml
```yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: ctlplane
  annotations:
    argocd.argoproj.io/sync-wave: "4"
spec:
  addresses:
  - 172.22.0.80-172.22.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: internalapi
  annotations:
    argocd.argoproj.io/sync-wave: "4"
spec:
  addresses:
  - 172.17.0.80-172.17.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: storage
  annotations:
    argocd.argoproj.io/sync-wave: "4"
spec:
  addresses:
  - 172.18.0.80-172.18.0.90
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: tenant
  annotations:
    argocd.argoproj.io/sync-wave: "4"
spec:
  addresses:
  - 172.19.0.80-172.19.0.90
```

### 4.2 L2Advertisement 리소스 구성

~/osp-ng-metal-lb-l2-advertisements.yaml
```yaml
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: ctlplane
  namespace: metallb-system
  annotations:
    argocd.argoproj.io/sync-wave: "3"
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
  annotations:
    argocd.argoproj.io/sync-wave: "3"
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
  annotations:
    argocd.argoproj.io/sync-wave: "3"
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
  annotations:
    argocd.argoproj.io/sync-wave: "3"
spec:
  ipAddressPools:
  - tenant
  interfaces:
  - enp1s0.22
```
<br>

## 5. 데이터-플레인 네트워크 구성

~/osp-ng-dataplane-netconfig.yaml
```yaml
kind: NetConfig
metadata:
  name: openstacknetconfig
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "5"
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
* NetConfig 커스터 리소스는 데이터-플레인 네트워크의 모든 서브넷을 구성
* 데이터-플레인에 적어도 하나의 컨트롤-플레인 네트워크를 정의해야 함
* VLAN 네트워크를 정의하여 InternalAPI, Storage, External과 같은 구성 가능한 네트워크에 대한 네트워크 격리를 생성 가능
* 각 네트워크 정의에는 IP 주소 할당을 포함
<br>

## 6. ArgoCD를 통해 오픈스택 네트워크 격리 구성

### 6.1 ArgoCD 애플리케이션 생성

ArgoCD 앱을 위한 YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: network-configuration
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/network-configuration
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: false
    syncOptions:
    - CreateNamespace=true
```

앱 생성
```bash
cat << EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: network-configuration
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/network-configuration
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: false
    syncOptions:
    - CreateNamespace=true
EOF
```
* ArgoCD 콘솔에 접속하여 네트워크 구성 배포 확인

### 6.2 네트워크 백엔드 확인 및 구성

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

### 6.3 NNCP 구성 확인

RHOSO의 네트워크 격리를 구성하기 위해 사용되는 NNCP (Node Network Configuration Policy) 리소스 확인
```bash
oc get nncp
oc describe nncp osp-enp1s0-worker-ocp4-worker1
```

실행 결과
```

```


### 6.4 NAD 구성 확인

각 격리된 네트워크에 해당 네트워크에 서비스 포드를 연결하는 NetworkAttachmentDefinition(nad) 리소스 확인 
```bash
oc get Network-Attachment-Definitions -n openstack
```

실행 결과
```

```

InternalAPI의 NAD IP 주소 구성을 확인
```bash
oc describe Network-Attachment-Definitions internalapi -n openstack
```

실행 결과
```

```

### 6.5 MetalLB 구성 확인

격리된 네트워크 상의 내부 서비스 엔드포인트를 노출하기 위해 사용되며, 기본적으로 퍼블릭 서비스 엔드포인트가 오픈시프트의 라우트로 노출됨

MetalLB의 IP 주소 범위 확인
```bash
oc get IPAddressPools -n metallb-system
```

실행 결과
```

```

미리 구성된 로컬 네트워크에 서비스를 광고하는 노드를 정의하는 L2Advertisement 리소스 확인
```bash
oc get L2Advertisements -n metallb-system
```

실행 결과
```

```

### 6.6 데이터-플레인 네트워크 확인

데이터-플레인 네트워크를 위해 모든 서브넷을 구성하는 NetConfig 커스텀 리소스(CR) 확인
```bash
oc get netconfigs -n openstack
oc describe netconfig openstacknetconfig -n openstack
```

실행 결과
```

```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 오퍼레이터 설치 <<](./install-oso-operators-via-argocd.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 컨트롤-플레인 생성 >>](./create-oso-ctl-plane-via-argocd.md)