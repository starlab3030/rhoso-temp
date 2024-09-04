# 오픈스택 서비스에 보안 접근 제공

목차
1. [오픈스택 모듈별 보안 접근을 위한 시크릿 생성](./provide-secure-access-to-rhoso.md#1-오픈스택-모듈별-보안-접근을-위한-시크릿-생성)<br>
2. [libvirt 서비스 보안 접근을 위한 시크릿 생성](./provide-secure-access-to-rhoso.md#2-libvirt-서비스-보안-접근을-위한-시크릿-생성)<br>
3. [요약](./provide-secure-access-to-rhoso.md#요약)<br>
<br>

## 1. 오픈스택 모듈별 보안 접근을 위한 시크릿 생성

오픈스택 각 모듈의 보안 접근을 위해 별도의 base64 패스워드가 필요합니다.

오픈스택 모듈별 암호 설정 YAML 파일
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: osp-secret
  namespace: openstack
type: Opaque
data:
  AdminPassword: b3BlbnN0YWNr
  CeilometerPassword: b3BlbnN0YWNr
  CinderDatabasePassword: b3BlbnN0YWNr
  CinderPassword: b3BlbnN0YWNr
  DatabasePassword: b3BlbnN0YWNr
  DbRootPassword: b3BlbnN0YWNr
  DesignateDatabasePassword: b3BlbnN0YWNr
  DesignatePassword: b3BlbnN0YWNr
  GlanceDatabasePassword: b3BlbnN0YWNr
  GlancePassword: b3BlbnN0YWNr
  HeatAuthEncryptionKey: b3BlbnN0YWNr
  HeatDatabasePassword: b3BlbnN0YWNr
  HeatPassword: b3BlbnN0YWNr
  IronicDatabasePassword: b3BlbnN0YWNr
  IronicInspectorDatabasePassword: b3BlbnN0YWNr
  IronicInspectorPassword: b3BlbnN0YWNr
  IronicPassword: b3BlbnN0YWNr
  KeystoneDatabasePassword: b3BlbnN0YWNr
  ManilaDatabasePassword: b3BlbnN0YWNr
  ManilaPassword: b3BlbnN0YWNr
  MetadataSecret: b3BlbnN0YWNr
  NeutronDatabasePassword: b3BlbnN0YWNr
  NeutronPassword: b3BlbnN0YWNr
  NovaAPIDatabasePassword: b3BlbnN0YWNr
  NovaAPIMessageBusPassword: b3BlbnN0YWNr
  NovaCell0DatabasePassword: b3BlbnN0YWNr
  NovaCell0MessageBusPassword: b3BlbnN0YWNr
  NovaCell1DatabasePassword: b3BlbnN0YWNr
  NovaCell1MessageBusPassword: b3BlbnN0YWNr
  NovaPassword: b3BlbnN0YWNr
  OctaviaDatabasePassword: b3BlbnN0YWNr
  OctaviaPassword: b3BlbnN0YWNr
  PlacementDatabasePassword: b3BlbnN0YWNr
  PlacementPassword: b3BlbnN0YWNr
  SwiftPassword: b3BlbnN0YWNr
```

오픈스택 모듈별 보안 접근을 위한 시크릿 생성
```bash
oc create -f osp-ng-ctlplane-secret.yaml
oc describe secret osp-secret -n openstack
```

출력 결과
```
[root@ocp4-bastion files]# oc create -f osp-ng-ctlplane-secret.yaml 
secret/osp-secret created

[root@ocp4-bastion files]# oc describe secret osp-secret -n openstack
Name:         osp-secret
Namespace:    openstack
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
HeatPassword:                     9 bytes
NovaAPIMessageBusPassword:        9 bytes
PlacementPassword:                9 bytes
ManilaPassword:                   9 bytes
NovaCell0MessageBusPassword:      9 bytes
SwiftPassword:                    9 bytes
IronicDatabasePassword:           9 bytes
NeutronPassword:                  9 bytes
NovaCell0DatabasePassword:        9 bytes
NovaCell1DatabasePassword:        9 bytes
OctaviaPassword:                  9 bytes
HeatDatabasePassword:             9 bytes
IronicInspectorPassword:          9 bytes
IronicPassword:                   9 bytes
CeilometerPassword:               9 bytes
CinderPassword:                   9 bytes
DesignatePassword:                9 bytes
HeatAuthEncryptionKey:            9 bytes
KeystoneDatabasePassword:         9 bytes
ManilaDatabasePassword:           9 bytes
NovaCell1MessageBusPassword:      9 bytes
DesignateDatabasePassword:        9 bytes
GlancePassword:                   9 bytes
IronicInspectorDatabasePassword:  9 bytes
MetadataSecret:                   9 bytes
NovaPassword:                     9 bytes
NeutronDatabasePassword:          9 bytes
NovaAPIDatabasePassword:          9 bytes
OctaviaDatabasePassword:          9 bytes
AdminPassword:                    9 bytes
CinderDatabasePassword:           9 bytes
DatabasePassword:                 9 bytes
DbRootPassword:                   9 bytes
GlanceDatabasePassword:           9 bytes
PlacementDatabasePassword:        9 bytes

[root@ocp4-bastion files]# 
```
<br>

## 2. libvirt 서비스 보안 접근을 위한 시크릿 생성

libvirt를 위한 시크릿을 생성하는 YAML 파일
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: libvirt-secret
  namespace: openstack
type: Opaque
data:
  LibvirtPassword: b3BlbnN0YWNr
```

libvirt를 위한 시크릿 생성
```bash
oc create -f osp-ng-libvirt-secret.yaml
oc describe secret libvirt-secret -n openstack
```

출력 결과
```
[root@ocp4-bastion files]# oc create -f osp-ng-libvirt-secret.yaml
secret/libvirt-secret created

[root@ocp4-bastion files]# oc describe secret libvirt-secret -n openstack
Name:         libvirt-secret
Namespace:    openstack
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
LibvirtPassword:  9 bytes

[root@ocp4-bastion files]# 
```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 오퍼레이터 설치 <<](./install-oso-operators.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 네트워크 구성 >>](./prepare-openshfit-for-rhoso-network-isolation.md)
