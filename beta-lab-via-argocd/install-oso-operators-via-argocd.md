# ArgoCD를 통한 오픈스택 플랫폼 서비스 오퍼레이터 설치

목차
1. [파일 리포지토리 복제](./install-oso-operators.md#1-파일-리포지토리를-복제)<br>
2. [오픈스택 오퍼레이터를 위한 프로젝트 생성](./install-oso-operators.md#2-오픈스택-오퍼레이터를-위한-프로젝트-생성)<br>
3. [오픈스택 서비스 배포를 위한 프로젝트 생성](./install-oso-operators.md#3-배포될-rhoso-환경을-위한-프로젝트-생성)<br>
4. [오퍼레이터 설치를 위한 시크릿 준비](./install-oso-operators.md#4-오퍼레이터-설치를-위한-시크릿-준비)<br>
5. [오퍼레이터 그룹 서브스크립션 준비](./install-oso-operators.md#5-카탈로그-소스-오퍼레이터-그룹-서브스크립션-준비)<br>
6. [오픈스택 오퍼레이터 설치 확인](./install-oso-operators.md#6-openstack-operatoropenstack-operators가-설치된-것을-확인)<br>
7. [오픈스택 오퍼레이터 포드 확인](./install-oso-operators.md#7-openstack-operators-네임스페이스에서-포드-확인)<br>
8. [요약](./install-oso-operators.md#요약)<br>
<br>

## 1. RHOSO 오퍼레이터 개요

RHOSO 서비스는 레드햇 오픈스프트 클러스터에서 실행되는 오퍼레이터 컬렉션으로 구현
* 이러한 오퍼레이터는 RHOSO 클라우드의 컴퓨팅, 스토리지, 네트워킹 및 기타 서비스를 관리
 
OpenStack Operator(openstack-operator)
* 필요한 모든 RHOSO 서비스 Operator를 설치하고 해당 Operator를 관리하는 데 사용하는 인터페이스

RHOSO 환경의 기본 아키텍처가 포함하는 기능
* 컨테이너 기반 애플리케이션 제공
  - 컨테이너 기반 RHOSO 배포를 제공하기 위해 레드햇 오픈시프트 및 RHEL 플랫폼에 걸쳐 있는 컨테이너 기반 접근 방식을 사용하여 제공
* 레드햇 오픈시프트 호스팅 서비스
  - 레드햇 오픈시프트는 오퍼레이터를 사용하여 수명 주기 관리를 제공하여 인프라 서비스와 RHOSO 컨트롤러 서비스를 호스팅
* Ansible 관리 RHEL 호스팅 서비스
  - RHOSO 워크로드는 전담 DataPlane 오퍼레이터가 관리하는 RHEL 노드에서 실행
  - DataPlane 오퍼레이터 Ansible 작업을 실행하여 Compute 노드와 같은 RHEL 데이터 플레인 노드를 구성
  - 레드햇 오픈시프트는 프로비저닝, DNS 및 구성 관리를 관리
* 설치자 프로비저닝 인프라 (IPI: Installer-Provisioned Infrastructure)
  - RHOSO installer는 RHOSO 베어메탈 머신 관리를 사용하여 RHOSO 클라우드에 대한 Compute 노드를 프로비저닝하는 설치자 프로비저닝 인프라(IPI)를 활성화
* 사용자 프로비저닝 인프라 (UPI: User-Provisioned Infrastructure)
  - 자체 머신 수집 및 프로비저닝 워크플로가 있는 경우 RHOSO 사전 프로비저닝 모델을 사용하여 컨테이너 기반 워크플로의 이점을 받으면서 사전 프로비저닝된 하드웨어를 RHOSO 환경에 추가
* 호스팅된 RHOSO 클라이언트
  - RHOSO는 배포된 RHOSO 환경에 대한 관리자 액세스 권한이 미리 구성된 호스트 openstackclient 포드를 제공
<br>

## 2. 오픈스택 오퍼레이터 설치 준비

### 2.1 오픈스택 오퍼레이터 설치를 위한 프로젝트 생성 준비

~/openstack-operators-namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openstack-operators
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

### 2.2 오픈스택 오퍼레이터 그룹 생성

~/openstack-operators-operatorgroup.yaml
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openstack
  namespace: openstack-operators
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

### 2.3 오픈스택 오퍼레이터를 위한 시크릿 준비

~/openstack-operators-secret.yaml
```yaml
apiVersion: v1
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJyZWdpc3RyeS5yZWRoYXQuaW8iOiB7CgkJCSJhdXRoIjogIk5qTTBNREExTm54dmMzQXRiMjR0YjJOd0xXeGlNVE0zTkRwbGVVcG9Za2RqYVU5cFNsTlZlbFY0VFdsS09TNWxlVXA2WkZkSmFVOXBTVEZaTWtWNlRUSk9hazVIV1RST1YwMHdUVzFhYlZsVVNUTlpiVlUxV1RKVmVVMVhTVE5OTWtwcVRVTktPUzVIUVhoblp6WklkREp2UTFNNGVuaElaSGRSZHpsclUwUTJVa2hsVVU5WFdXRkVUMk51VVVJMVVrVnNaWGRSUzNaYWJXTk9WMmt0V1Vwa1NXNUtOV2xZVkVVNWNqbDBSMVpKVGpkbWFFWktURGRtTFdob1RERlFTekpTVm5wYVNFUTRjWGxtYTAxWFkwTkZSalZIVlhad09ISkVXRFJIUkhKVGEzRnFjRlZFTkRSMFpWZFphMDk1T1U1aUxUTndUMGQ2VWtsRE4zRnpPRGgxVTNoTmVqZG9abWxzTkVsZlNHMXFSalJCUVZCSmFUUnFNMUZhYUhBd2JIRnlXSHA2WmpkMmREWk9UR3hwZWtSR1lUSllWR05RWmw5MlVYRlNaVVoxTTBGZk5XbFhabms0V0cxTWJFTTNVVWxwZUdWV2RqSkpSUzFoYUZKeFRWOVZSRU5tTlVSbk0yNHlWM0JaZG0xUU5XcGpVMUJHVDB4dlZEZHpUV2x0ZVdWaFVFSnVZVGM1TTJKdmFWZ3ljM2R0WlVkSVVUSXpkSGd4YmtaaGRrTlZZWFpIZGw5alJGSkJkbnBXV0VOS01rNVNUMVJLTlhWdVNHbE9OME5ZUldKNmJUUlNaeTAyTlhSWk5FUXdXWGx1VkZVNFREWjBNR2RaZEZoWldWazVYM2RwTVhoT2N5MWpVMmhCYlVOTmFERjVVMHB1T1c1Q1kzRTBlV1IyU0RkbFVXNW9VMFYyYjBzd1lsQnpUbDkyVjBwRFowOVJRbEY1VDJSd1ZHWlNUVlUyY0dsQmVUbElNWHBLTUV0NmMxTjZkVXRUVXpobVdEQnRPVzlPTjI1aGNscFFiRE0wUkZScFJWVlVSR1ZYT0Y5VFV6WjJTbXBJY2w5Uk9VOWZXRFJ0Vm1WbFVXaElNbTlqVGw4MFRUbFNOa0U0T1hSdFVUSnFUMkoxVjIwdFkzVXhXV3N0UnpaR1UxQlZUMDVvYzI5RFh6azVibEZ1U1VOVE5HMUJkVU5YVjBSSWVFWlpOakZvU1hKeVpWWmFRbE5JTURVelRXZG1VMkZITW5OeFZHSXlOazFyZUV0WGVDMVVVREZ6ZURFNGNHSXhlRzF2TkVsUlJYZEpURWxpVEd4VFVFRXpkbUZtWW5KaVVVODFVbEZqYlROVldVdDBXWGRsZGpCMlFXeE1OWFJoV0dsVWRVeEZlVkJ6WTJSNmRqQlRZdz09IgoJCX0KCX0KfQ==
kind: Secret
metadata:
  name: osp-operators-secret
  namespace: openstack-operators
  annotations:
    argocd.argoproj.io/sync-wave: "0"
type: kubernetes.io/dockerconfigjson
```

### 2.4 오퍼레이터 오퍼레이터 그룹 서브스크립션 준비

~/openstack-operators-subscription.yaml
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/openstack-operator.openstack-operators: ""
  name: openstack-operator
  namespace: openstack-operators
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  channel: stable-v1.0
  installPlanApproval: Automatic
  name: openstack-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: openstack-operator.v1.0.0
```
<br>

## 3. ArgoCD를 통한 오픈스택 오퍼레이터 설치 및 구성

### 3.1 오픈스택 오퍼레이터 설치

오픈스택 오퍼레이터 설치 YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openstack-operators-installation
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/openstack-operators-installation
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

오픈스택 오퍼레이터 설치
```bash
cat << EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openstack-operators-installation
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/openstack-operators-installation
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

### 3.2 오픈스택 오퍼레이터 설치 확인

openstack-operators 네임스페이스 상의 포드 확인
```bash
oc get pods -n openstack-operators
```

실행 결과
```

```

완료된 포드 및 실행 중인 오픈스택 서비스 포드 확인
```bash
oc get pods -n openstack-operators --sort-by=.metadata.creationTimestamp
```

실행 결과
```

```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 사전 준비 <<](./pre-requisite-ops-via-argocd.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 네트워크 구성 >>](./prepare-openshfit-for-rhoso-network-isolation-via-argocd.md)
