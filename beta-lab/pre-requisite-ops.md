# 오픈스택 서비스 설치를 위한 사전 준비

목차
1. [개요](./pre-requisite-ops.md#1-개요)<br>
2. [베이스천 접속 및 오픈시프트 클러스터 확인](./pre-requisite-ops.md#2-베이스천-접속-및-오픈시프트-클러스터-확인)<br>
3. [NMState 오퍼레이터 설치](./pre-requisite-ops.md#3-nmstate-오퍼레이터-설치)<br>
4. [MetalLB 오퍼레이터 설치](./pre-requisite-ops.md#4-metallb-오퍼레이터-설치)<br>
5. [Cert-Manager 오퍼레이터 설치](./pre-requisite-ops.md#5-cert-manager-오퍼레이터-설치)<br>
6. [요약](./pre-requisite-ops.md#요약)<br>
<br>

## 1. 개요

공통 준비 사항은 다음과 같습니다.
* 오픈시프트 클러스터가 MULTUS CNI를 지원
* oc / podman 명령어 라인 도구 지원

오픈스택 서비스 오퍼레이터를 설치하기 전에 다음 3가지 오퍼레이터가 준비되어 있어야 합니다.
* NMState 오퍼레이터
* MetalLB 오퍼레이터
* Cert-Manager 오퍼레이터
<br>

## 2. 베이스천 접속 및 오픈시프트 클러스터 확인

*hypervisor* 노드에서 *bastion* 노드로 SSH 접속
```bash
sudo -i ssh root@192.168.123.100
```

오픈시프트 클러스터 노드 확인
```bash
oc get nodes
```

출력 결과
```
[root@ocp4-bastion ~]# oc get nodes
NAME                           STATUS   ROLES                  AGE   VERSION
ocp4-master1.aio.example.com   Ready    control-plane,master   18h   v1.28.11+add48d0
ocp4-master2.aio.example.com   Ready    control-plane,master   18h   v1.28.11+add48d0
ocp4-master3.aio.example.com   Ready    control-plane,master   18h   v1.28.11+add48d0
ocp4-worker1.aio.example.com   Ready    worker                 17h   v1.28.11+add48d0
ocp4-worker2.aio.example.com   Ready    worker                 17h   v1.28.11+add48d0
ocp4-worker3.aio.example.com   Ready    worker                 17h   v1.28.11+add48d0

[root@ocp4-bastion ~]# 
```
<br>

## 3. NMState 오퍼레이터 설치

### 3.1 **nmstate** 오퍼레이터 네임스페이스 생성

생성 YAML 파일
```yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: openshift-nmstate
    name: openshift-nmstate
  name: openshift-nmstate
spec:
  finalizers:
  - kubernetes
```

오퍼레이터 네임스페이스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: openshift-nmstate
    name: openshift-nmstate
  name: openshift-nmstate
spec:
  finalizers:
  - kubernetes
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

EOF
namespace/openshift-nmstate created

[root@ocp4-bastion ~]# oc get namespace/openshift-nmstate
NAME                STATUS   AGE
openshift-nmstate   Active   96s

[root@ocp4-bastion ~]#
```
<br>

### 3.2 **OperatorGroup** 생성

오퍼러에터 그룹 생성 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  annotations:
    olm.providedAPIs: NMState.v1.nmstate.io
  name: openshift-nmstate
  namespace: openshift-nmstate
spec:
  targetNamespaces:
  - openshift-nmstate
```

오퍼레이터 그룹 생성
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  annotations:
    olm.providedAPIs: NMState.v1.nmstate.io
  name: openshift-nmstate
  namespace: openshift-nmstate
spec:
  targetNamespaces:
  - openshift-nmstate
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

EOF
operatorgroup.operators.coreos.com/openshift-nmstate created

[root@ocp4-bastion ~]# oc get operatorgroup -n openshift-nmstate
NAME                AGE
openshift-nmstate   57s

[root@ocp4-bastion ~]# 
```
<br>

### 3.3 **nmstate** 오퍼레이터를 구독

서브스크립션 등록 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/kubernetes-nmstate-operator.openshift-nmstate: ""
  name: kubernetes-nmstate-operator
  namespace: openshift-nmstate
spec:
  channel: stable
  installPlanApproval: Automatic
  name: kubernetes-nmstate-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

서브스크립션 등록
```bash
cat << EOF| oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/kubernetes-nmstate-operator.openshift-nmstate: ""
  name: kubernetes-nmstate-operator
  namespace: openshift-nmstate
spec:
  channel: stable
  installPlanApproval: Automatic
  name: kubernetes-nmstate-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF| oc apply -f -

...<snip>...

EOF
subscription.operators.coreos.com/kubernetes-nmstate-operator created

[root@ocp4-bastion ~]# oc get clusterserviceversion -n openshift-nmstate -o custom-columns=Name:.metadata.name,Phase:.status.phase
Name                                              Phase
kubernetes-nmstate-operator.4.15.0-202408211438   Succeeded

[root@ocp4-bastion ~]# 
```
* *Phase*가 *Succeeded*가 된 것을 확인
<br>

### 3.4 **nmstate** 오퍼레이터 인스턴스 생성

인스턴스 생성 YAML 파일
```yaml
apiVersion: nmstate.io/v1
kind: NMState
metadata:
  name: nmstate
```

인스턴스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: nmstate.io/v1
kind: NMState
metadata:
  name: nmstate
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -
...<snip>...
nmstate.nmstate.io/nmstate created

[root@ocp4-bastion ~]# oc get clusterserviceversion -n openshift-nmstate \
 -o custom-columns=Name:.metadata.name,Phase:.status.phase
Name                                              Phase
kubernetes-nmstate-operator.4.15.0-202408211438   Succeeded

[root@ocp4-bastion ~]# 
```
* *Phase*가 *Succeeded*인 것을 확인
<br>

## 4. MetalLB 오퍼레이터 설치

### 4.1 **MetalLB** 오퍼레이터 네임스페이스 생성

생성 YAML 파일
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
```

**metallb-system** 네임스페이스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
EOF
namespace/metallb-system created

[root@ocp4-bastion ~]# oc get namespace/metallb-system
NAME             STATUS   AGE
metallb-system   Active   37s

[root@ocp4-bastion ~]# 
```
<br>

### 4.2 **OperatorGroup** 생성

오퍼레이터 그룹 생성 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: metallb-operator
  namespace: metallb-system
```

오퍼레이터 그룹 생성
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: metallb-operator
  namespace: metallb-system
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

operatorgroup.operators.coreos.com/metallb-operator created

[root@ocp4-bastion ~]# oc get operatorgroup -n metallb-system
NAME               AGE
metallb-operator   19s

[root@ocp4-bastion ~]# 
```
<br>

### 4.3  **metalb** 오퍼레이터 서브스크립션 등록

서브스크립션 등록 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: metallb-system
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

서브스크립션 등록
```bash
cat << EOF| oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: metallb-system
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF| oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: metallb-system
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
subscription.operators.coreos.com/metallb-operator-sub created

[root@ocp4-bastion ~]# oc get installplan -n metallb-system
NAME            CSV                                     APPROVAL    APPROVED
install-z4h6v   metallb-operator.v4.15.0-202408210608   Automatic   true

[root@ocp4-bastion ~]# oc get clusterserviceversion -n metallb-system  -o custom-columns=Name:.metadata.name,Phase:.status.phase
Name                                    Phase
metallb-operator.v4.15.0-202408210608   Installing

[root@ocp4-bastion ~]# oc get clusterserviceversion -n metallb-system  -o custom-columns=Name:.metadata.name,Phase:.status.phase
Name                                    Phase
metallb-operator.v4.15.0-202408210608   Succeeded

[root@ocp4-bastion ~]# 
```
* *Phase*가 *Succeeded*가 된 것을 확인
<br>

### 4.4 **metallb** 인스턴스 생성

한 개의 인스턴스를 생성하는 YAML 파일
```yaml
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
```

인스턴스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: metallb.io/v1beta1
kind: MetalLB
metadata:
  name: metallb
  namespace: metallb-system
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

metallb.metallb.io/metallb created

[root@ocp4-bastion ~]# oc get deployment -n metallb-system controller
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
controller   0/1     1            0           11s

[root@ocp4-bastion ~]# oc get deployment -n metallb-system controller
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
controller   1/1     1            1           22s

[root@ocp4-bastion ~]# oc get daemonset -n metallb-system speaker
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                            AGE
speaker   3         3         3       3            3           kubernetes.io/os=linux,node-role.kubernetes.io/worker=   33s

[root@ocp4-bastion ~]#
```
* 배포 후 *AVAILABLE*이 1이 될 때까지 기다림
* 이후 *daemonset*에서 **speaker**가 실행 중임을 확인
<br>

## 5. Cert-Manager 오퍼레이터 설치

### 5.1 **cert-manager** 네임스페이스 생성

생성 YAML 파일
```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: cert-manager-operator
    labels:
      pod-security.kubernetes.io/enforce: privileged
      security.openshift.io/scc.podSecurityLabelSync: "false"
```

네임스페이스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
    name: cert-manager-operator
    labels:
      pod-security.kubernetes.io/enforce: privileged
      security.openshift.io/scc.podSecurityLabelSync: "false"
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

namespace/cert-manager-operator created

[root@ocp4-bastion ~]# oc get namespace/cert-manager-operator
NAME                    STATUS   AGE
cert-manager-operator   Active   45s

[root@ocp4-bastion ~]#
```
<br>

### 5.2 **OperatorGroup** 생성

오퍼레이터 그룹 생성 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cert-manager-operator
  namespace: cert-manager-operator
spec:
  targetNamespaces:
  - cert-manager-operator
  upgradeStrategy: Default
```

오퍼레이터 그룹 생성
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cert-manager-operator
  namespace: cert-manager-operator
spec:
  targetNamespaces:
  - cert-manager-operator
  upgradeStrategy: Default
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

operatorgroup.operators.coreos.com/cert-manager-operator created

[root@ocp4-bastion ~]# oc get operatorgroup -n cert-manager-operator
NAME                    AGE
cert-manager-operator   15s
[root@ocp4-bastion ~]# 
```
<br>

### 5.3 **cert-manager** 서브스크립션 등록

서브스크립션 등록 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/openshift-cert-manager-operator.cert-manager-operator: ""
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
spec:
  channel: stable-v1
  installPlanApproval: Automatic
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```

서브스크립션 등록
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/openshift-cert-manager-operator.cert-manager-operator: ""
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
spec:
  channel: stable-v1
  installPlanApproval: Automatic
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

출력 결과
```
[root@ocp4-bastion ~]# cat << EOF | oc apply -f -

...<snip>...

subscription.operators.coreos.com/openshift-cert-manager-operator created

[root@ocp4-bastion ~]# oc get installplan -n cert-manager-operator
NAME            CSV                             APPROVAL    APPROVED
install-btg8j   cert-manager-operator.v1.14.0   Automatic   true

[root@ocp4-bastion ~]# oc get clusterserviceversion -n cert-manager-operator \
 -o custom-columns=Name:.metadata.name,Phase:.status.phase
Name                                    Phase
cert-manager-operator.v1.14.0           Installing
metallb-operator.v4.15.0-202408210608   Succeeded

[root@ocp4-bastion ~]# oc get pods -n cert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-78dc65d766-24wdp             1/1     Running   0          38s
cert-manager-cainjector-57d5dd7bb-77gtk   1/1     Running   0          74s
cert-manager-webhook-55b954fb87-fxd2l     1/1     Running   0          77s

[root@ocp4-bastion ~]# 
```
* *Phase*가 *Succeeded*인 것을 확인
* *cert-manager* 관련 포드가 실행 중인 것을 확인
<br>

## 요약 

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 오퍼레이터 설치 >>](./install-oso-operators.md)
