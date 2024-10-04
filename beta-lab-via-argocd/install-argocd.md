# GipOps를 위한 ArgoCD 준비 및 설치

## 1. ArgoCD 설치

### 1.1 오퍼레이터 네임스페이스 생성

ArgoCD 오퍼레이터를 위핸 네임스페이스 생성 YAML 파일
```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: openshift-gitops-operator
    labels:
      pod-security.kubernetes.io/enforce: privileged
      security.openshift.io/scc.podSecurityLabelSync: "false"
```

오퍼레이터를 위한 네임스페이스 생성
```bash
cat << EOF | oc apply -f -
apiVersion: v1
kind: Namespace
metadata:
    name: openshift-gitops-operator
    labels:
      pod-security.kubernetes.io/enforce: privileged
      security.openshift.io/scc.podSecurityLabelSync: "false"
EOF
```

실행 결과
```

```
<br>

## 1.2 오퍼레이터 그룹 생성

오퍼레이터 그룹 생성을 위한 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  generateName: openshift-gitops-operator-
  name: openshift-gitops-operator-b8wcv
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
```

오퍼레이터 그룹 생성
```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  generateName: openshift-gitops-operator-
  name: openshift-gitops-operator-b8wcv
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
EOF
```

실행 결과
```
# oc get operatorgroup -n openshift-gitops-operator

#
```
<br>

## 1.3 Argocd 오퍼레이터에 등록

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generation: 1
  labels:
    operators.coreos.com/openshift-gitops-operator.openshift-gitops-operator: ""
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: latest
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: openshift-gitops-operator.v1.12.0
```

```bash
cat << EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  generation: 1
  labels:
    operators.coreos.com/openshift-gitops-operator.openshift-gitops-operator: ""
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: latest
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: openshift-gitops-operator.v1.12.0
EOF
```

실행 결과
```

```
<br>

## 1.4 오퍼레이터 운영중인지 확인

명령어를 실행하여 오퍼레이터가 운영 중인지 확인
```bash
oc get clusterserviceversion -n openshift-gitops-operator -o custom-columns=Name:.metadata.name,Phase:.status.phase -w
```
* *Phase* 필드가 *Succeeded*로 나오는 지 확인

실행 결과
```

```
<br>

## 1.5 ArogCD를 위한 ServiceAccount에 클러스터를 관리 권한 설정

```bash
oc adm policy add-cluster-role-to-user cluster-admin -z openshift-gitops-argocd-application-controller -n openshift-gitops
```

실행 결과
```

```
<br>

## 1.6 오픈시프트 GitOps에 연결하여, 기본 admin 관리자와 첫 번째 배포 후의 랜덤 패스워스 생성

admin 사용자의 시크릿에서 패스워드 추출
```bash
argoPass=$(oc get secret/openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d)
echo $argoPass
```

실행 결과
```

```
<br>

## 1.7 오픈시프트 GitOps의 라우트(route)를 얻기

```bash
argoURL=$(oc get route openshift-gitops-server -n openshift-gitops -o jsonpath='{.spec.host}{"\n"}')
echo $argoURL
```

실행 결과
```

```
* 오픈시프트 GitOps 콘솔에 사용자 이름(admin)과 패스워드로 로그인
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 사전 준비 >>](./pre-requisite-ops-via-argocd.md)

