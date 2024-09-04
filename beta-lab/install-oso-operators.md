# 오픈스택 플랫폼 서비스 오퍼레이터 설치

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

## 1. 파일 리포지토리를 복제

랩에서 사용할 리포지토리를 복제하고, 해당 디렉토리로 이동 합니다.
```bash
git clone https://github.com/rh-osp-demo/dp-demo.git
cd $HOME/labrepo/content/files
```

출력 결과
```
[root@ocp4-bastion ~]# git clone https://github.com/rh-osp-demo/dp-demo.git
Cloning into 'labrepo'...
remote: Enumerating objects: 573, done.
remote: Counting objects: 100% (141/141), done.
remote: Compressing objects: 100% (77/77), done.
remote: Total 573 (delta 77), reused 95 (delta 46), pack-reused 432 (from 1)
Receiving objects: 100% (573/573), 1012.64 KiB | 26.65 MiB/s, done.
Resolving deltas: 100% (344/344), done.

[root@ocp4-bastion ~]# cd $HOME/labrepo/content/files

[root@ocp4-bastion files]# 
```
<br>

## 2. 오픈스택 오퍼레이터를 위한 프로젝트 생성

오픈스택 오퍼레이터 설치를 위한 프로젝트 **openstack-operators**를 생성
```bash
oc new-project openstack-operators
```

출력 결과
```
[root@ocp4-bastion files]# oc new-project openstack-operators
Now using project "openstack-operators" on server "https://api.ocp.example.com:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[root@ocp4-bastion files]# oc get project/openstack-operators
NAME                  DISPLAY NAME   STATUS
openstack-operators                  Active

[root@ocp4-bastion files]# 
```
<br>

## 3. 배포될 RHOSO 환경을 위한 프로젝트 생성

배포될 오픈스택 환경을 위한 프로젝트 생성
```bash
oc new-project openstack
```

출력 결과
```
[root@ocp4-bastion files]# oc new-project openstack
Now using project "openstack" on server "https://api.ocp.example.com:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[root@ocp4-bastion files]# oc get project/openstack
NAME        DISPLAY NAME   STATUS
openstack                  Active

[root@ocp4-bastion files]# 
```
<br>

## 4. 오퍼레이터 설치를 위한 시크릿 준비

### 4.1 레지스트리에 로그인

*podman* 명령어를 이용하여 레지스트리 *registry.redhat.io*에 로그인
```bash
podman login --username "6340056|osp-on-ocp-lb1374" --password "eyJhbGciOiJSUzUxMiJ9.eyJzdWIiOiI1Y2EzM2NjNGY4NWM0MmZmYTI3YmU5Y2UyMWI3M2JjMCJ9.GAxgg6Ht2oCS8zxHdwQw9kSD6RHeQOWYaDOcnQB5RElewQKvZmcNWi-YJdInJ5iXTE9r9tGVIN7fhFJL7f-hhL1PK2RVzZHD8qyfkMWcCEF5GUvp8rDX4GDrSkqjpUD44teWYkOy9Nb-3pOGzRIC7qs88uSxMz7hfil4I_HmjF4AAPIi4j3QZhp0lqrXzzf7vt6NLlizDFa2XTcPf_vQqReFu3A_5iWfy8XmLlC7QIixeVv2IE-ahRqM_UDCf5Dg3n2WpYvmP5jcSPFOLoT7sMimyeaPBna793boiX2swmeGHQ23tx1nFavCUavGv_cDRAvzVXCJ2NROTJ5unHiN7CXEbzm4Rg-65tY4D0YynTU8L6t0gYtXYYY9_wi1xNs-cShAmCMh1ySJn9nBcq4ydvH7eQnhSEvoK0bPsN_vWJCgOQBQyOdpTfRMU6piAy9H1zJ0KzsSzuKSS8fX0m9oN7narZPl34DTiEUTDeW8_SS6vJjHr_Q9O_X4mVeeQhH2ocN_4M9R6A89tmQ2jObuWm-cu1Yk-G6FSPUONhsoC_99nQnICS4mAuCWWDHxFY61hIrreVZBSH053MgfSaG2sqTb26MkxKWx-TP1sx18pb1xmo4IQEwILIbLlSPA3vafbrbQO5RQcm3UYKtYwev0vAlL5taXiTuLEyPscdzv0Sc" registry.redhat.io --authfile auth.json
```

출력 결과
```
[root@ocp4-bastion files]# podman login --username "6340056|osp-on-ocp-lb1374" --password "eyJhbGciOiJSUzUxMiJ9.eyJzdWIiOiI1Y2EzM2NjNGY4NWM0Mm ...<snip>... 3UYKtYwev0vAlL5taXiTuLEyPscdzv0Sc" registry.redhat.io --authfile auth.json
Login Succeeded!

[root@ocp4-bastion files]#
```
<br>

### 4.2 시크릿 생성

레지스트리를 위한 시크릿 생성
```bash
oc create secret generic osp-operators-secret \
 -n openstack-operators \
 --from-file=.dockerconfigjson=auth.json \
 --type=kubernetes.io/dockerconfigjson
```

출력 결과
```
[root@ocp4-bastion files]# oc create secret generic osp-operators-secret \
 -n openstack-operators \
 --from-file=.dockerconfigjson=auth.json \
 --type=kubernetes.io/dockerconfigjson
secret/osp-operators-secret created

[root@ocp4-bastion files]# oc get secret/osp-operators-secret -n openstack-operators
NAME                   TYPE                             DATA   AGE
osp-operators-secret   kubernetes.io/dockerconfigjson   1      117s

[root@ocp4-bastion files]# 
```
<br>

## 5. 카탈로그 소스, 오퍼레이터 그룹, 서브스크립션 준비

오픈스택 오퍼레이터를 위한 커스텀-리소스(CR)를 생성하는 YAML 파일
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: openstack-operator-index
  namespace: openstack-operators
spec:
  sourceType: grpc
  secrets:
    - osp-operators-secret
  gprcPodConfig:
    securityContextConfig: legacy
  image: quay.io/redhat_emp1/pnavarro-beta-openstack-operator-index:latest
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openstack
  namespace: openstack-operators
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openstack-operator
  namespace: openstack-operators
spec:
  name: openstack-operator
  channel: alpha
  source: openstack-operator-index
  sourceNamespace: openstack-operators
```

오픈스택 오퍼레이터 생성
```bash
oc apply -f osp-ng-openstack-operator.yaml
```

출력 결과
```
[root@ocp4-bastion files]# oc apply -f osp-ng-openstack-operator.yaml
catalogsource.operators.coreos.com/openstack-operator-index created
operatorgroup.operators.coreos.com/openstack created
subscription.operators.coreos.com/openstack-operator created

[root@ocp4-bastion files]# oc get catalogsource -n openstack-operators
NAME                       DISPLAY   TYPE   PUBLISHER   AGE
openstack-operator-index             grpc               2m16s

[root@ocp4-bastion files]# oc get operatorgroup -n openstack-operators
NAME        AGE
openstack   43s

[root@ocp4-bastion files]# oc get subscription/openstack-operator -n openstack-operators
NAME                 PACKAGE              SOURCE                     CHANNEL
openstack-operator   openstack-operator   openstack-operator-index   alpha

[root@ocp4-bastion files]# 
```
<br>

## 6. **openstack-operator.openstack-operators**가 설치된 것을 확인

오픈스택 오퍼레이터 설치 후에, **openstack-operator.openstack-operators**가 설치 되었는 지 확인
```bash
oc get operators/openstack-operator.openstack-operators -n openstack-operators
```

출력 결과
```
[root@ocp4-bastion files]# oc get operators/openstack-operator.openstack-operators -n openstack-operators
NAME                                     AGE
openstack-operator.openstack-operators   6m16s

[root@ocp4-bastion files]# 
```
<br>

## 7. **openstack-operators** 네임스페이스에서 포드 확인

### 7.1 오픈스택 오퍼레이터 포드 리스트 확인

포드 리스트 확인
```bash
oc get pods -n openstack-operators
```

출력 결과
```
[root@ocp4-bastion files]# oc get pods -n openstack-operators
NAME                                                              READY   STATUS      RESTARTS   AGE
0aa4542351b81c759f5d66d2a27bb21c6e8c6ad182d5e0bc5b26143d47df9jc   0/1     Completed   0          11m
0c40fcce6a29f383cda66e9e9de41fdbe4494f3db489c33c0d8a452de0vst2j   0/1     Completed   0          11m
0cb7980427badb1a09887f737b6e7926293c96df07677613cd4a3e5a099n6tf   0/1     Completed   0          11m
0d5e8b8ece0e0a19b70bbacbe14ecf17e61d8dee28fc2f0f957129d790z88h8   0/1     Completed   0          11m
1d370dd1211d9f8cd7941e6027f281bf64a935395acf9fa989f2a4727ecj9pv   0/1     Completed   0          11m
3f5d6b27b81f5083069a18b338ed754ff082fed8db5aa226f5085721a85p4qg   0/1     Completed   0          11m
663e6d8ad4f196bf54db8672da19bb2f5c813658a2be92f0d49bffb61478b66   0/1     Completed   0          11m
6bd41423d2e2562598f62eb8adeb83df97de37905b4bd9c16523bf0a0767dvf   0/1     Completed   0          11m
71eb50d904d797d5aaac6402dcb8985b10f9ab63b8216f03c0e63e2b9827tkb   0/1     Completed   0          11m
7c94bb3b02ca9042c5f6e14c65952ef3c67cca033056c88c5e9b26384bzxnxl   0/1     Completed   0          11m
7c9fad0d480e0c828983facb45c31ed2e155e882cecd508872abb69cc766mn6   0/1     Completed   0          11m
83548d653efd60937228b24b616096095d1a95e6336917c9d47e67e9balh4mg   0/1     Completed   0          11m
87b653f8f46ca35a737a68f84d38e68072ac4175bc92e03f3186710447nq8wr   0/1     Completed   0          11m
b49dd4a931e45136a1939039b80f61c625439b182b80c8da49ceaac696rbk97   0/1     Completed   0          11m
barbican-operator-controller-manager-8557cc6b8d-2nrm9             2/2     Running     0          10m
bb2c8487079b923caf36d46d512857bdad1766f5c1da648718aca2914a7v6nt   0/1     Completed   0          11m
bf4a22f9f279a3cb9fbb8203b93fc5afef3c4139a7047cdfdefa76a9f6bnj6l   0/1     Completed   0          11m
c72e9eaf2cbf7d5d56afeceee16eb0a22202dd1b57d8e17d13d3f315b0qq2ch   0/1     Completed   0          11m
cinder-operator-controller-manager-65f888dcb8-27sxn               2/2     Running     0          10m
d99ee5e400f0504e500dab0c14e8a0c64ef081923513a2e1d36b0012bb8tzn2   0/1     Completed   0          11m
dataplane-operator-controller-manager-766d47f8c7-k7n9m            2/2     Running     0          9m2s
ddb73e2f52e24b33c181cfd4e79d641b53fc4d05d25a1c4b2023fb29b6t4zjj   0/1     Completed   0          11m
designate-operator-controller-manager-67fccc68b-6n9qv             2/2     Running     0          9m33s
f6f52fde99b131818d1d7f0d389d5fea6e7e59bf76cf2cc7fe59979c5dbqbbg   0/1     Completed   0          11m
fa70a9bfe037af640412c3d89f5c5a1c436e067203446c86880d43de9bxqw7z   0/1     Completed   0          11m
fe9e1162edff42ab6306d5db9a347c5ba1210b0ad753db597a6cd18accqqvjm   0/1     Completed   0          11m
glance-operator-controller-manager-57ffd4cdb5-dg84m               2/2     Running     0          9m45s
heat-operator-controller-manager-798dcfb956-hwvjs                 2/2     Running     0          11m
horizon-operator-controller-manager-fcfcdf8c8-4wfd4               2/2     Running     0          10m
infra-operator-controller-manager-6bdb57c4bc-2bkbh                2/2     Running     0          9m23s
ironic-operator-controller-manager-5c788c67db-fmxcd               2/2     Running     0          9m37s
keystone-operator-controller-manager-cfd89c484-6rlqh              2/2     Running     0          9m15s
manila-operator-controller-manager-5559bdd979-sns8x               2/2     Running     0          9m47s
mariadb-operator-controller-manager-589d879cb9-4t76x              2/2     Running     0          9m54s
neutron-operator-controller-manager-94d9b6968-p55rr               2/2     Running     0          9m52s
nova-operator-controller-manager-5b56ff4d7d-c6ncj                 2/2     Running     0          9m41s
octavia-operator-controller-manager-75d868b49c-c9sxd              2/2     Running     0          9m40s
openstack-ansibleee-operator-controller-manager-d658658ff-8nw44   2/2     Running     0          9m19s
openstack-baremetal-operator-controller-manager-76ddb86c5db526h   2/2     Running     0          9m34s
openstack-operator-controller-manager-dcd588d5c-l672b             2/2     Running     0          9m3s
openstack-operator-index-zf55z                                    1/1     Running     0          11m
ovn-operator-controller-manager-5f797b84c-2zs4m                   2/2     Running     0          10m
placement-operator-controller-manager-d495d7f59-sbbwg             2/2     Running     0          11m
rabbitmq-cluster-operator-544cbfcd5c-v2724                        1/1     Running     0          9m56s
swift-operator-controller-manager-7f8c8f58bb-gb2fl                2/2     Running     0          10m
telemetry-operator-controller-manager-56cf6b58fb-zf72k            2/2     Running     0          10m

[root@ocp4-bastion files]#
```
<br>

### 7.2 오픈스택의 완료 및 실행 중인 포드 확인

완료 및 실행 중인 포드 리스트 확인
```bash
oc get pods -n openstack-operators --sort-by=.metadata.creationTimestamp
```

출력 결과
```
[root@ocp4-bastion files]# oc get pods -n openstack-operators --sort-by=.metadata.creationTimestamp
NAME                                                              READY   STATUS      RESTARTS   AGE
openstack-operator-index-zf55z                                    1/1     Running     0          12m
0c40fcce6a29f383cda66e9e9de41fdbe4494f3db489c33c0d8a452de0vst2j   0/1     Completed   0          11m
663e6d8ad4f196bf54db8672da19bb2f5c813658a2be92f0d49bffb61478b66   0/1     Completed   0          11m
d99ee5e400f0504e500dab0c14e8a0c64ef081923513a2e1d36b0012bb8tzn2   0/1     Completed   0          11m
c72e9eaf2cbf7d5d56afeceee16eb0a22202dd1b57d8e17d13d3f315b0qq2ch   0/1     Completed   0          11m
bf4a22f9f279a3cb9fbb8203b93fc5afef3c4139a7047cdfdefa76a9f6bnj6l   0/1     Completed   0          11m
83548d653efd60937228b24b616096095d1a95e6336917c9d47e67e9balh4mg   0/1     Completed   0          11m
3f5d6b27b81f5083069a18b338ed754ff082fed8db5aa226f5085721a85p4qg   0/1     Completed   0          11m
6bd41423d2e2562598f62eb8adeb83df97de37905b4bd9c16523bf0a0767dvf   0/1     Completed   0          11m
0d5e8b8ece0e0a19b70bbacbe14ecf17e61d8dee28fc2f0f957129d790z88h8   0/1     Completed   0          11m
1d370dd1211d9f8cd7941e6027f281bf64a935395acf9fa989f2a4727ecj9pv   0/1     Completed   0          11m
fa70a9bfe037af640412c3d89f5c5a1c436e067203446c86880d43de9bxqw7z   0/1     Completed   0          11m
fe9e1162edff42ab6306d5db9a347c5ba1210b0ad753db597a6cd18accqqvjm   0/1     Completed   0          11m
b49dd4a931e45136a1939039b80f61c625439b182b80c8da49ceaac696rbk97   0/1     Completed   0          11m
f6f52fde99b131818d1d7f0d389d5fea6e7e59bf76cf2cc7fe59979c5dbqbbg   0/1     Completed   0          11m
87b653f8f46ca35a737a68f84d38e68072ac4175bc92e03f3186710447nq8wr   0/1     Completed   0          11m
0cb7980427badb1a09887f737b6e7926293c96df07677613cd4a3e5a099n6tf   0/1     Completed   0          11m
bb2c8487079b923caf36d46d512857bdad1766f5c1da648718aca2914a7v6nt   0/1     Completed   0          11m
ddb73e2f52e24b33c181cfd4e79d641b53fc4d05d25a1c4b2023fb29b6t4zjj   0/1     Completed   0          11m
7c94bb3b02ca9042c5f6e14c65952ef3c67cca033056c88c5e9b26384bzxnxl   0/1     Completed   0          11m
0aa4542351b81c759f5d66d2a27bb21c6e8c6ad182d5e0bc5b26143d47df9jc   0/1     Completed   0          11m
7c9fad0d480e0c828983facb45c31ed2e155e882cecd508872abb69cc766mn6   0/1     Completed   0          11m
71eb50d904d797d5aaac6402dcb8985b10f9ab63b8216f03c0e63e2b9827tkb   0/1     Completed   0          11m
heat-operator-controller-manager-798dcfb956-hwvjs                 2/2     Running     0          11m
placement-operator-controller-manager-d495d7f59-sbbwg             2/2     Running     0          11m
cinder-operator-controller-manager-65f888dcb8-27sxn               2/2     Running     0          11m
ovn-operator-controller-manager-5f797b84c-2zs4m                   2/2     Running     0          11m
telemetry-operator-controller-manager-56cf6b58fb-zf72k            2/2     Running     0          11m
swift-operator-controller-manager-7f8c8f58bb-gb2fl                2/2     Running     0          10m
barbican-operator-controller-manager-8557cc6b8d-2nrm9             2/2     Running     0          10m
horizon-operator-controller-manager-fcfcdf8c8-4wfd4               2/2     Running     0          10m
rabbitmq-cluster-operator-544cbfcd5c-v2724                        1/1     Running     0          10m
mariadb-operator-controller-manager-589d879cb9-4t76x              2/2     Running     0          10m
neutron-operator-controller-manager-94d9b6968-p55rr               2/2     Running     0          10m
manila-operator-controller-manager-5559bdd979-sns8x               2/2     Running     0          9m59s
glance-operator-controller-manager-57ffd4cdb5-dg84m               2/2     Running     0          9m57s
nova-operator-controller-manager-5b56ff4d7d-c6ncj                 2/2     Running     0          9m53s
octavia-operator-controller-manager-75d868b49c-c9sxd              2/2     Running     0          9m52s
ironic-operator-controller-manager-5c788c67db-fmxcd               2/2     Running     0          9m49s
openstack-baremetal-operator-controller-manager-76ddb86c5db526h   2/2     Running     0          9m46s
designate-operator-controller-manager-67fccc68b-6n9qv             2/2     Running     0          9m45s
infra-operator-controller-manager-6bdb57c4bc-2bkbh                2/2     Running     0          9m35s
openstack-ansibleee-operator-controller-manager-d658658ff-8nw44   2/2     Running     0          9m31s
keystone-operator-controller-manager-cfd89c484-6rlqh              2/2     Running     0          9m27s
openstack-operator-controller-manager-dcd588d5c-l672b             2/2     Running     0          9m15s
dataplane-operator-controller-manager-766d47f8c7-k7n9m            2/2     Running     0          9m14s

[root@ocp4-bastion files]# 
```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 사전 준비 <<](./pre-requisite-ops.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 보안 접근 구성 >>](./provide-secure-access-to-rhoso.md)
