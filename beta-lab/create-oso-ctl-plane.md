# 오픈스택 서비스의 컨트롤-플레인 생성

목차
1. [오픈스택 백엔드 스토리지로 NFS 준비](./create-oso-ctl-plane.md#1-오픈스택-서비스의-스토리지를-위한-nfs-준비)<br>
2. [NFS 스토리지 클래스 구성](./create-oso-ctl-plane.md#2-nfs-스토리지-클래스-구성)<br>
3. [스토리지 백엔드를 위한 시크릿 준비](./create-oso-ctl-plane.md#3-스토리지-백엔드를-위한-시크릿-준비)<br>
4. [오픈스택 컨트롤-플레인 설치 준비](./create-oso-ctl-plane.md#4-오픈스택-서비스-컨트롤-플레인-설치-준비)<br>
5. [오픈스택 컨트롤-플레인 설치](./create-oso-ctl-plane.md#5-오픈스택-서비스-컨트롤-플레인-설치)<br>
6. [요약](./create-oso-ctl-plane.md#요약)<br>
<br>

## 1. 오픈스택 서비스의 스토리지를 위한 NFS 준비

### 1.1 베이스천 노드에서 NFS 서비스 확인

베이스천 노드의 NFS 서비스 확인 및 공유된 디렉터리 확인
```bash
systemctl status nfs-server
exportfs
showmount -e
```

출력 결과
```
[root@ocp4-bastion ~]# systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Wed 2024-08-28 01:24:57 EDT; 1 day 17h ago
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 1081 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 1092 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 1256 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssprox>
   Main PID: 1256 (code=exited, status=0/SUCCESS)
        CPU: 29ms

Aug 28 01:24:57 ocp4-bastion.aio.example.com systemd[1]: Starting NFS server and services...
Aug 28 01:24:57 ocp4-bastion.aio.example.com systemd[1]: Finished NFS server and services.

[root@ocp4-bastion ~]# exportfs
/nfs            <world>

[root@ocp4-bastion ~]# showmount -e
Export list for ocp4-bastion.aio.example.com:
/nfs *

[root@ocp4-bastion ~]#
```
<br>

### 1.2 Cinder를 위한 NFS 공유 만들기

Cinder 백엔드 볼륨을 위한 NFS 공유 만들기
```bash
mkdir -pv /nfs/cinder
chmod 777 /nfs/cinder/
ls -ld /nfs/cinder/
```

출력 결과
```
[root@ocp4-bastion ~]# mkdir -pv /nfs/cinder
mkdir: created directory '/nfs/cinder'

[root@ocp4-bastion ~]# chmod 777 /nfs/cinder/

[root@ocp4-bastion ~]# ls -ld /nfs/cinder/
drwxrwxrwx. 2 root root 6 Aug 29 18:38 /nfs/cinder/

[root@ocp4-bastion ~]#
```
<br>

## 2. NFS 스토리지 클래스 구성

### 2.1 NFS 스토리지 클래스 YAML 파일 확인

```bash
yq -y '.' $HOME/labrepo/content/files/nfs-storage.yaml
```

NFS 스토리지 클래스를 생성하는 YAML 파일
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv1
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv1
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv2
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv2
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv3
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv3
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv4
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv4
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv5
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv5
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv6
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv6
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv7
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv7
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv8
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv8
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv9
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv9
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv10
spec:
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  capacity:
    storage: 100Gi
  nfs:
    path: /nfs/pv10
    server: 192.168.123.100
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs
  volumeMode: Filesystem
```
* /nfs/pv1 ~ pv10까지 있는 것을 확인
<br>

### 2.2 디렉터리 생성

NFS 스토리지 클래스를 위해, NFS 상에 공유 디렉터리 생성
```bash
for i in {6..11}; do mkdir -pv /nfs/pv$i; done
chmod 777 /nfs/pv*
ls -ld /nfs/pv*
```
* 디렉터리 생성은 /nfs/pv6 ~ pv11까지 생성하는 것을 확인

출력 결과
```
[root@ocp4-bastion ~]# for i in {6..11}; do mkdir -pv /nfs/pv$i; done
mkdir: created directory '/nfs/pv6'
mkdir: created directory '/nfs/pv7'
mkdir: created directory '/nfs/pv8'
mkdir: created directory '/nfs/pv9'
mkdir: created directory '/nfs/pv10'
mkdir: created directory '/nfs/pv11'

[root@ocp4-bastion ~]# chmod 777 /nfs/pv*

[root@ocp4-bastion ~]# ls -ld /nfs/pv*
drwxrwxrwx. 2 root root 6 Aug 28 01:22 /nfs/pv1
drwxrwxrwx. 2 root root 6 Aug 29 19:05 /nfs/pv10
drwxrwxrwx. 2 root root 6 Aug 29 19:08 /nfs/pv11
drwxrwxrwx. 2 root root 6 Aug 28 01:22 /nfs/pv2
drwxrwxrwx. 2 root root 6 Aug 28 01:22 /nfs/pv3
drwxrwxrwx. 2 root root 6 Aug 28 01:22 /nfs/pv4
drwxrwxrwx. 2 root root 6 Aug 28 01:22 /nfs/pv5
drwxrwxrwx. 2 root root 6 Aug 29 19:05 /nfs/pv6
drwxrwxrwx. 2 root root 6 Aug 29 19:05 /nfs/pv7
drwxrwxrwx. 2 root root 6 Aug 29 19:05 /nfs/pv8
drwxrwxrwx. 2 root root 6 Aug 29 19:05 /nfs/pv9

[root@ocp4-bastion ~]#
```
<br>

### 2.3 NFS 스토리지 클래스 생성

오픈시프트 상에 NFS 스토리지 클래서 생성
```bash
oc create -f $HOME/labrepo/content/files/nfs-storage.yaml
oc get storageclass/nfs
oc get pv -o json | jq -r '.items[]|select(.spec.storageClassName|match("nfs"))|.metadata.name'
```

출력 결과
```
[root@ocp4-bastion ~]# oc create -f $HOME/labrepo/content/files/nfs-storage.yaml
storageclass.storage.k8s.io/nfs created
persistentvolume/nfs-pv1 created
persistentvolume/nfs-pv2 created
persistentvolume/nfs-pv3 created
persistentvolume/nfs-pv4 created
persistentvolume/nfs-pv5 created
persistentvolume/nfs-pv6 created
persistentvolume/nfs-pv7 created
persistentvolume/nfs-pv8 created
persistentvolume/nfs-pv9 created
persistentvolume/nfs-pv10 created

[root@ocp4-bastion ~]# oc get storageclass/nfs
NAME   PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs    kubernetes.io/no-provisioner   Delete          Immediate           false                  33s

[root@ocp4-bastion ~]# oc get pv -o json | jq -r '.items[]|select(.spec.storageClassName|match("nfs"))|.metadata.name'
nfs-pv1
nfs-pv10
nfs-pv2
nfs-pv3
nfs-pv4
nfs-pv5
nfs-pv6
nfs-pv7
nfs-pv8
nfs-pv9

[root@ocp4-bastion ~]#
```
<br>

## 3. 스토리지 백엔드를 위한 시크릿 준비

### 3.1 Cinder와 NFS 서버 연결을 위한 시크릿 준비

```bash
cat $HOME/labrepo/content/files/nfs-cinder-conf
oc create secret generic cinder-nfs-config --from-file=$HOME/labrepo/content/files/nfs-cinder-conf
oc get secret/cinder-nfs-config
```

출력 결과
```
[root@ocp4-bastion ~]# cat $HOME/labrepo/content/files/nfs-cinder-conf
[nfs]
nas_host=192.168.123.100
nas_share_path=/nfs/cinder

[root@ocp4-bastion ~]# oc create secret generic cinder-nfs-config --from-file=$HOME/labrepo/content/files/nfs-cinder-conf
secret/cinder-nfs-config created

[root@ocp4-bastion ~]# oc get secret/cinder-nfs-config
NAME                TYPE     DATA   AGE
cinder-nfs-config   Opaque   1      16s

[root@ocp4-bastion ~]#
```
<br>

### 3.2 Glance와 NFS 서버 연결을 위한 시크릿 준비

```bash
cat $HOME/labrepo/content/files/glance-conf
oc create secret generic glance-cinder-config --from-file=$HOME/labrepo/content/files/glance-conf
oc get secret/glance-cinder-config
```

출력 결과
```
[root@ocp4-bastion ~]# cat $HOME/labrepo/content/files/glance-conf
[default_backend]
cinder_store_user_name = glance
cinder_store_password = openstack
cinder_store_project_name = service

[root@ocp4-bastion ~]# oc create secret generic glance-cinder-config --from-file=$HOME/labrepo/content/files/glance-conf
secret/glance-cinder-config created

[root@ocp4-bastion ~]# oc get secret/glance-cinder-config
NAME                   TYPE     DATA   AGE
glance-cinder-config   Opaque   1      26s

[root@ocp4-bastion ~]#
```
<br>

## 4. 오픈스택 서비스 컨트롤-플레인 설치 준비

### 4.1 오픈스택 컨트롤-플레인 구성 파일 준비

```bash
yq -y '.' $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
```

오픈스택 서비스 전체 구성 파일
```yaml
apiVersion: core.openstack.org/v1beta1
kind: OpenStackControlPlane
metadata:
  name: openstack-galera-network-isolation
spec:
  secret: osp-secret
  storageClass: ocs-storagecluster-ceph-rbd
  tls:
    podLevel:
      enabled: false
  dns:
    template:
      override:
        service:
          metadata:
            annotations:
              metallb.universe.tf/address-pool: ctlplane
              metallb.universe.tf/allow-shared-ip: ctlplane
              metallb.universe.tf/loadBalancerIPs: 172.22.0.89
          spec:
            type: LoadBalancer
      options:
        - key: server
          values:
            - 192.168.123.100
      replicas: 1
  cinder:
    apiOverride:
      route: {}
    template:
      databaseInstance: openstack
      secret: osp-secret
      cinderAPI:
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
      cinderScheduler:
        replicas: 1
      cinderBackup:
        networkAttachments:
          - storage
        replicas: 0
      cinderVolumes:
        nfs:
          networkAttachments:
            - storage
          customServiceConfig: '[nfs]

            volume_backend_name=nfs

            volume_driver=cinder.volume.drivers.nfs.NfsDriver

            nfs_snapshot_support=true

            nas_secure_file_operations=false

            nas_secure_file_permissions=false

            '
          customServiceConfigSecrets:
            - cinder-nfs-config
  barbican:
    enabled: false
    apiOverride:
      route: {}
    template:
      databaseInstance: openstack
      secret: osp-secret
      barbicanAPI:
        replicas: 1
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
      barbicanWorker:
        replicas: 1
      barbicanKeystoneListener:
        replicas: 1
  glance:
    apiOverrides:
      default:
        route: {}
    template:
      customServiceConfig: '[DEFAULT]

        enabled_backends = default_backend:cinder

        [glance_store]

        default_backend = default_backend

        [default_backend]

        rootwrap_config = /etc/glance/rootwrap.conf

        description = Default cinder backend

        cinder_catalog_info volumev3::publicURL

        '
      customServiceConfigSecrets:
        - glance-cinder-config
      databaseInstance: openstack
      storageClass: nfs
      storageRequest: 10G
      secret: osp-secret
      keystoneEndpoint: default
      glanceAPIs:
        default:
          type: single
          replicas: 1
          override:
            service:
              internal:
                metadata:
                  annotations:
                    metallb.universe.tf/address-pool: internalapi
                    metallb.universe.tf/allow-shared-ip: internalapi
                    metallb.universe.tf/loadBalancerIPs: 172.17.0.80
                spec:
                  type: LoadBalancer
          networkAttachments:
            - storage
  keystone:
    apiOverride:
      route: {}
    template:
      override:
        service:
          internal:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.80
            spec:
              type: LoadBalancer
      databaseInstance: openstack
      secret: osp-secret
  galera:
    templates:
      openstack:
        storageRequest: 500M
      openstack-cell1:
        storageRequest: 500M
  memcached:
    templates:
      memcached:
        replicas: 1
  neutron:
    apiOverride:
      route: {}
    template:
      override:
        service:
          internal:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.80
            spec:
              type: LoadBalancer
      databaseInstance: openstack
      secret: osp-secret
      networkAttachments:
        - internalapi
  horizon:
    apiOverride:
      route: {}
    template:
      replicas: 1
      secret: osp-secret
  nova:
    apiOverride:
      route: {}
    template:
      apiServiceTemplate:
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
      metadataServiceTemplate:
        override:
          service:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.80
            spec:
              type: LoadBalancer
      secret: osp-secret
  manila:
    enabled: false
    apiOverride:
      route: {}
    template:
      manilaAPI:
        replicas: 1
        networkAttachments:
          - internalapi
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
      manilaScheduler:
        replicas: 1
      manilaShares:
        share1:
          replicas: 1
          networkAttachments:
            - storage
  ovn:
    template:
      ovnDBCluster:
        ovndbcluster-nb:
          dbType: NB
          storageRequest: 10G
          networkAttachment: internalapi
        ovndbcluster-sb:
          dbType: SB
          storageRequest: 10G
          networkAttachment: internalapi
      ovnController:
        networkAttachment: tenant
  placement:
    apiOverride:
      route: {}
    template:
      override:
        service:
          internal:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.80
            spec:
              type: LoadBalancer
      databaseInstance: openstack
      secret: osp-secret
  rabbitmq:
    templates:
      rabbitmq:
        override:
          service:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.85
            spec:
              type: LoadBalancer
      rabbitmq-cell1:
        override:
          service:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.86
            spec:
              type: LoadBalancer
  heat:
    apiOverride:
      route: {}
    cnfAPIOverride:
      route: {}
    enabled: false
    template:
      databaseInstance: openstack
      heatAPI:
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
        replicas: 1
      heatEngine:
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
        replicas: 1
      secret: osp-secret
  ironic:
    enabled: false
    template:
      databaseInstance: openstack
      ironicAPI:
        replicas: 1
      ironicConductors:
        - replicas: 1
          storageRequest: 10G
      ironicInspector:
        replicas: 1
      ironicNeutronAgent:
        replicas: 1
      secret: osp-secret
  telemetry:
    enabled: true
    template:
      metricStorage:
        enabled: false
        monitoringStack:
          alertingEnabled: true
          scrapeInterval: 30s
          storage:
            strategy: persistent
            retention: 24h
            persistent:
              pvcStorageRequest: 20G
      autoscaling:
        enabled: false
        aodh:
          passwordSelectors: null
          databaseAccount: aodh
          databaseInstance: openstack
          secret: osp-secret
        heatInstance: heat
      ceilometer:
        enabled: true
        secret: osp-secret
      logging:
        enabled: false
        network: internalapi
        ipaddr: 172.17.0.80
        port: 10514
        cloNamespace: openshift-logging
  swift:
    enabled: false
    proxyOverride:
      route: {}
    template:
      swiftRing:
        ringReplicas: 1
      swiftStorage:
        replicas: 1
        networkAttachments:
          - storage
      swiftProxy:
        replicas: 1
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
        networkAttachments:
          - storage
  octavia:
    enabled: false
    template:
      databaseInstance: openstack
      octaviaAPI:
        replicas: 1
      secret: osp-secret
  designate:
    enabled: false
    apiOverride:
      route: {}
    template:
      databaseInstance: openstack
      secret: osp-secret
      designateAPI:
        override:
          service:
            internal:
              metadata:
                annotations:
                  metallb.universe.tf/address-pool: internalapi
                  metallb.universe.tf/allow-shared-ip: internalapi
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.80
              spec:
                type: LoadBalancer
      designateCentral:
        replicas: 1
      designateWorker:
        replicas: 0
        networkAttachments:
          - designate
      designateProducer:
        replicas: 0
      designateMdns:
        replicas: 0
        networkAttachments:
          - designate
      designateBackendbind9:
        replicas: 0
        networkAttachments:
          - designate
```
<br>

### 4.2 모듈별 구성 확인

#### 4.2.1 Cinder 구성

```bash
yq -y '.spec.cinder' $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
```

```yaml
apiOverride:
  route: {}
template:
  databaseInstance: openstack
  secret: osp-secret
  cinderAPI:
    override:
      service:
        internal:
          metadata:
            annotations:
              metallb.universe.tf/address-pool: internalapi
              metallb.universe.tf/allow-shared-ip: internalapi
              metallb.universe.tf/loadBalancerIPs: 172.17.0.80
          spec:
            type: LoadBalancer
  cinderScheduler:
    replicas: 1
  cinderBackup:
    networkAttachments:
      - storage
    replicas: 0
  cinderVolumes:
    nfs:
      networkAttachments:
        - storage
      customServiceConfig: '[nfs]

        volume_backend_name=nfs

        volume_driver=cinder.volume.drivers.nfs.NfsDriver

        nfs_snapshot_support=true

        nas_secure_file_operations=false

        nas_secure_file_permissions=false

        '
      customServiceConfigSecrets:
        - cinder-nfs-config
```

#### 4.2.2 Glance 구성

```bash
yq -y '.spec.glance' $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
```

```yaml
apiOverrides:
  default:
    route: {}
template:
  customServiceConfig: '[DEFAULT]

    enabled_backends = default_backend:cinder

    [glance_store]

    default_backend = default_backend

    [default_backend]

    rootwrap_config = /etc/glance/rootwrap.conf

    description = Default cinder backend

    cinder_catalog_info volumev3::publicURL

    '
  customServiceConfigSecrets:
    - glance-cinder-config
  databaseInstance: openstack
  storageClass: nfs
  storageRequest: 10G
  secret: osp-secret
  keystoneEndpoint: default
  glanceAPIs:
    default:
      type: single
      replicas: 1
      override:
        service:
          internal:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.80
            spec:
              type: LoadBalancer
      networkAttachments:
        - storage
```

#### 4.2.3 Keystone 구성

```bash
yq -y '.spec.keystone' $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
```

```yaml
apiOverride:
  route: {}
template:
  override:
    service:
      internal:
        metadata:
          annotations:
            metallb.universe.tf/address-pool: internalapi
            metallb.universe.tf/allow-shared-ip: internalapi
            metallb.universe.tf/loadBalancerIPs: 172.17.0.80
        spec:
          type: LoadBalancer
  databaseInstance: openstack
  secret: osp-secret
```

#### 4.2.4 Neutron 구성

```bash
yq -y '.spec.neutron' $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
```

```yaml
apiOverride:
  route: {}
template:
  override:
    service:
      internal:
        metadata:
          annotations:
            metallb.universe.tf/address-pool: internalapi
            metallb.universe.tf/allow-shared-ip: internalapi
            metallb.universe.tf/loadBalancerIPs: 172.17.0.80
        spec:
          type: LoadBalancer
  databaseInstance: openstack
  secret: osp-secret
  networkAttachments:
    - internalapi
```

#### 4.2.5 

```bash

```

```yaml

```
<br>

## 5. 오픈스택 서비스 컨트롤-플레인 설치

오픈스택 컨트롤-플레인 설치 시작
```bash
oc create -f $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
oc get openstackcontrolplane -n openstack
```

출력 결과
```
[root@ocp4-bastion ~]# oc create -f $HOME/labrepo/content/files/osp-ng-ctlplane-deploy.yaml
openstackcontrolplane.core.openstack.org/openstack-galera-network-isolation created

[root@ocp4-bastion ~]# oc get openstackcontrolplane -n openstack
NAME                                 STATUS   MESSAGE
openstack-galera-network-isolation   False    OpenStackControlPlane RabbitMQ in progress

[root@ocp4-bastion ~]# oc get openstackcontrolplane -n openstack
NAME                                 STATUS   MESSAGE
openstack-galera-network-isolation   False    OpenStackControlPlane OVN in progress

...<snip>...

[root@ocp4-bastion ~]# oc get openstackcontrolplane -n openstack
NAME                                 STATUS   MESSAGE
openstack-galera-network-isolation   True     Setup complete

[root@ocp4-bastion ~]#
```
* 설치는 몇 분이 소요됨
* *MESSAGE*에서 *Setup complete*이 나타나는 것을 확인
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 네트워크 구성 <<](./prepare-openshfit-for-rhoso-network-isolation.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 서비스 데이터-플레인 구성 >>](./configure-data-plane.md)