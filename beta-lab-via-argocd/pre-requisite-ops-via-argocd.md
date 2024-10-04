# 오픈스택 서비스 설치를 위한 사전 준비

목차
1. [NMState 오퍼레이터 설치 준비](./pre-requisite-ops-via-argocd.md#1-nmstate-오퍼레이터-설치-준비)<br>
2. [MetalLB 오퍼레이터 설치 준비](./pre-requisite-ops-via-argocd.md#2-metallb-오퍼레이터-설치-준비)<br>
3. [Cert-Manager 오퍼레이터 설치 준비](./pre-requisite-ops-via-argocd.md#3-cert-manager-오퍼레이터-설치-준비)<br>
4. [ArgoCD 앱을 사용하여 RHOSO 관련 사전 오퍼레이터 설치](./pre-requisite-ops-via-argocd.md#4-argocd-앱을-사용하여-rhoso-관련-사전-오퍼레이터-설치)<br>
5. [요약](./pre-requisite-ops.md#요약)<br>
<br>

## 1. NMState 오퍼레이터 설치 준비

**NMState** 오퍼레이터
* 오픈시프트 클러스터에서 노드 네트워크 구성을 관리하는 선언적 방법을 제공하는 쿠버네티스 오퍼레이터
* NNCP(Node Network Configuration Policy)를 사용하여 클러스터 노드에서 원하는 네트워크 상태를 구동
* NMState 프로젝트를 활용

* 주로 오픈스택 서비스 네트워크에 대한 네트워크 격리를 구성하기 위해 오픈시프트 노드를 구성할 때 사용
* 또한 Compute 노드에서 네트워킹을 구성하는 데에도 매우 유용

### 1.1 **nmstate** 오퍼레이터 네임스페이스

~/nmstate-namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-nmstate
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
spec:
  finalizers:
  - kubernetes
```

### 1.2 **nmstate** 오퍼레이터 그룹

~/nmstate-operatorgroups.yaml
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-nmstate
  namespace: openshift-nmstate
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  targetNamespaces:
  - openshift-nmstate
```

### 1.3 **nmstate** 오퍼레이터 서브스크립션

~/nmstate-subscription.yaml
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: kubernetes-nmstate-operator
  namespace: openshift-nmstate
  labels:
    operators.coreos.com/kubernetes-nmstate-operator.openshift-nmstate: ""
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  channel: stable
  installPlanApproval: Automatic
  name: kubernetes-nmstate-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
<br>

## 2. MetalLB 오퍼레이터 설치 준비 

**MetalLB** 오퍼레이터
* 베어 메탈 쿠버네티스 클러스터를 위한 로드 밸런서 구현으로, 표준 네트워크 장비와 통합되는 네트워크 로드 밸런싱 솔루션을 제공
* 클라우드 공급업체(AWS ELB, Google Cloud Load Balancer 등)가 제공하는 외부 로드 밸런서를 사용할 수 없는 환경에서 MetalLB는 로드 밸런싱 서비스를 제공

* RHOSO에서는 L2 계층 모드를 사용
* 이 모드에서 MetalLB는 서비스의 IP 주소에 대한 주소 확인 프로토콜(ARP) 요청에 응답하여 기본적으로 로컬 네트워크에서 IP 주소를 "클레임"
* 클라이언트가 해당 IP 주소로 트래픽을 보내면 MetalLB는 해당 트래픽을 서비스의 Pod 중 하나로 전달

### 2.1 **metallb** 오퍼레이터 네임스페이스

~/metallb-namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: metallb-system
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

### 2.2 **metallb** 오퍼레이터 그룹

~/metallb-operatorgroup.yaml
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: metallb-operator
  namespace: metallb-system
  annotations:
    argocd.argoproj.io/sync-wave: "0"
```

### 2.3 **metallb** 오퍼레이터 서브스크립션

~/metallb-subscription.yaml
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: metallb-operator-sub
  namespace: metallb-system
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  channel: stable
  name: metallb-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
<br>

## 3. Cert-Manager 오퍼레이터 설치 준비

**Cert-Manager** 오퍼레이터
* cert-manager를 통해, 쿠버네티스 애드온이 쿠버네티스 클러스터 내에서 TLS 인증서의 관리 및 발급을 자동화
* *Let's Encrypt*와 같은 공개 CA와 사설 CA를 포함하여 다양한 인증 기관(CA)에서 SSL/TLS 인증서를 획득, 갱신 및 사용하는 프로세스를 간소화
* cert-manager를 사용하면 사용자가 인증서 수명 주기를 수동으로 관리하지 않고도 HTTPS를 통해 애플리케이션이 안전하게 제공 가능

* RHOSO에서 cert-manager를 사용하여 TLS everywhere(TLS-e)라고 하는 오픈스택 엔드포인트에서 SSL/TLS를 활성화하기 위한 SSL/TLS 인증서를 발급하고 관리

### 3.1 **cert-manager** 오퍼레이터 네임스페이스

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-operator
  labels:
    pod-security.kubernetes.io/enforce: privileged
    security.openshift.io/scc.podSecurityLabelSync: "false"
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

### 3.2 **cert-manager** 오퍼레이터 그룹

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cert-manager-operator
  namespace: cert-manager-operator
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  targetNamespaces:
  - cert-manager-operator
  upgradeStrategy: Default
```

### 3.3 **cert-manager** 오퍼레이터 서브스크립션

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
  labels:
    operators.coreos.com/openshift-cert-manager-operator.cert-manager-operator: ""
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  channel: stable-v1
  installPlanApproval: Automatic
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
<br>

## 4. ArgoCD 앱을 사용하여 RHOSO 관련 사전 오퍼레이터 설치

### 4.1 ArgoCD 설정

RHOSO 컨트롤-플레인 배포를 위해, 사전에 필요한 오퍼레이터 설치를 위한 ArgoCD 앱 매니페스트를 생성합니다.

ArgoCD 설정을 위한 YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: pre-requisites-operators-installation
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/pre-reqs-operators
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
* GitOps로 사전 설치를 진행하기 위해 해당 Git URL에 필요한 파일의 위치를 입력

```bash
cat << EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: pre-requisites-operators-installation
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/pre-reqs-operators
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

실행 결과
```

```
* ArgoCD 콘솔에서 사전 준비를 위한 오퍼레이터 설치가 완료되는 것을 확인
<br>

### 4.2 NMState 오퍼레이터 설치 완료

#### 4.2.1 NMState 인스턴스 생성

```bash
cat << EOF | oc apply -f -
apiVersion: nmstate.io/v1
kind: NMState
metadata:
  name: nmstate
EOF
```

실행 결과
```

```

#### 4.2.2 CLI에서 NMState 포드 확인

```bash
oc get pods -n openshift-nmstate
```

실행 결과
```

```

### 4.3 MetalLB 오퍼레이터 설치 완료

#### 4.3.1 MetalLB 인스턴스 생성

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

실행 결과
```

```

#### 4.3.2 CLI에서 MetalLB 포드 확인

```bash
oc get pods -n metallb-system
```

실행 결과
```

```

### 4.4 Cert-Manager 오퍼레이터 설치 확인

#### 4.4.1 CLI에서 Cert-Manager 포드 확인

```bash
oc get pods -n cert-manager
```

실행 결과
```

```
<br>


## 요약 

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< ArgoCD 설치 <<](./install-argocd.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 오퍼레이터 설치 >>](./install-oso-operators-via-argocd.md)
