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

## 2. 데이터-플레인을 위한 가상머신 생성

### 2.1 hypervisor 노드 상에서 컴퓨트 가상머신 생성

root로 권한 상승 후 RHEL 컴퓨트 가상머신 생성
```bash
sudo -i

cd /var/lib/libvirt/images
cp rhel-9.4-x86_64-kvm.qcow2 rhel9-guest.qcow2
qemu-img info rhel9-guest.qcow2
qemu-img resize rhel9-guest.qcow2 +90G
chown -R qemu:qemu rhel9-*.qcow2

virt-customize -a rhel9-guest.qcow2 --run-command 'growpart /dev/sda 4'
virt-customize -a rhel9-guest.qcow2 --run-command 'xfs_growfs /'
virt-customize -a rhel9-guest.qcow2 --root-password password:redhat
virt-customize -a rhel9-guest.qcow2 --run-command 'systemctl disable cloud-init'
virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --ssh-inject root:file:/root/.ssh/id_rsa.pub
virt-customize -a /var/lib/libvirt/images/rhel9-guest.qcow2 --selinux-relabel

qemu-img create -f qcow2 -F qcow2 -b /var/lib/libvirt/images/rhel9-guest.qcow2 /var/lib/libvirt/images/osp-compute-0.qcow2

virt-install --virt-type kvm --ram 16384 --vcpus 4 --cpu=host-passthrough --os-variant rhel8.4 --disk path=/var/lib/libvirt/images/osp-compute-0.qcow2,device=disk,bus=virtio,format=qcow2 --network network:ocp4-provisioning --network network:ocp4-net --boot hd,network --noautoconsole --vnc --name osp-compute0 --noreboot

virsh start osp-compute0
```

실행 결과
```

```

### 2.2 컴퓨트 노드에 로그인 후 검증

#### 2.2.1 컴퓨트 노드의 IP 주소 확인

생성된 컴퓨트 노드의 IP 주소가 192.168.123.0/24 내에 위치하는 지 확인
```bash
watch virsh domifaddr osp-compute0 --source agent
```

실행 결과
```

```
* IP 주소 할당이 끌나면, eth1에 할당된 IP 주소를 사용하여 로그인

#### 2.2.2 컴퓨트 노드의 네트워크 구성

컴퓨트 노드에 암호 없이 SSH 로그인 후 네트워크 구성
```bash
nmcli co delete 'Wired connection 1'
nmcli con add con-name "static-eth0" ifname eth0 type ethernet ip4 172.22.0.100/24 ipv4.dns "172.22.0.89"
nmcli con up "static-eth0"
nmcli co delete 'Wired connection 2'
nmcli con add con-name "static-eth1" ifname eth1 type ethernet ip4 192.168.123.61/24 ipv4.dns "192.168.123.100" ipv4.gateway "192.168.123.1"
nmcli con up "static-eth1"
```

실행 결과
```

```

#### 2.2.3 로그아웃 후 SSH 키 설정

가지고 있는 SSH 키로 컴퓨트 노드의 키 설정
```bash
sudo -i
scp /root/.ssh/id_rsa root@192.168.123.100:/root/.ssh/id_rsa_compute
scp /root/.ssh/id_rsa.pub root@192.168.123.100:/root/.ssh/id_rsa_compute.pub
```

실행 결과
```

```

#### 2.2.4 데이터-플레인 앤서블 SSH 키 시크릿 생성

```bash
oc create secret generic dataplane-ansible-ssh-private-key-secret --save-config --dry-run=client --from-file=authorized_keys=/root/.ssh/id_rsa_compute.pub --from-file=ssh-privatekey=/root/.ssh/id_rsa_compute --from-file=ssh-publickey=/root/.ssh/id_rsa_compute.pub -n openstack -o yaml | oc apply -f-
```

실행 결과
```

```
<br>

## 3. 컨트롤-플레인 설치를 위한 구성 파일

### 3.1 Cinder 백엔드로서 NFS 구성을 위한 시크릿 생성

~/secret-cinder-nfs-config.yaml
```yaml
apiVersion: v1
data:
  nfs-cinder-conf: W25mc10KbmFzX2hvc3Q9MTkyLjE2OC4xMjMuMTAwCm5hc19zaGFyZV9wYXRoPS9uZnMvY2luZGVy
kind: Secret
metadata:
  name: cinder-nfs-config
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
type: Opaque
```

### 3.2 libvirt를 위한 시크릿 생성  

~/osp-ng-libvirt-secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: libvirt-secret
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "0"
type: Opaque
data:
  LibvirtPassword: b3BlbnN0YWNr
```

### 3.3 가상머신 마이그레이션을 위한 SSH 키 시크릿 생성

~/secret-nova-migration-ssh-key.yaml
```yaml
apiVersion: v1
data:
  ssh-privatekey: LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFyQUFBQUJObFkyUnpZUwoxemFHRXlMVzVwYzNSd05USXhBQUFBQ0c1cGMzUndOVEl4QUFBQWhRUUFaenZJa0R4NkRCNm9EOHFqMU5MRlZGb0xERFkrClZHT2VrVEdqUHlWbTNXNEk4cDE3VFE1RG9CK3lWWU5PeHJBcFk4KzFpU2hEMTBzenFYU2dnTHdVbllrQVJiWElOVVQxL0kKeWt2YUgzSzVNVm5NbGFXUEFPbGtpUzRybHpNR2ZtcDBNZGFHM0tyOFJxemEydGE4a29nOEE0ZHZVK0gyZnZEbVU4WU0zYQo3TjlVVlJBQUFBRWdQa1ZtSFQ1RlpoMEFBQUFUWldOa2MyRXRjMmhoTWkxdWFYTjBjRFV5TVFBQUFBaHVhWE4wY0RVeU1RCkFBQUlVRUFHYzd5SkE4ZWd3ZXFBL0tvOVRTeFZSYUN3dzJQbFJqbnBFeG96OGxadDF1Q1BLZGUwME9RNkFmc2xXRFRzYXcKS1dQUHRZa29ROWRMTTZsMG9JQzhGSjJKQUVXMXlEVkU5ZnlNcEwyaDl5dVRGWnpKV2xqd0RwWklrdUs1Y3pCbjVxZERIVwpodHlxL0VhczJ0cld2SktJUEFPSGIxUGg5bjd3NWxQR0ROMnV6ZlZGVVFBQUFBUWdFeVNPRHY2R0J0Uk1nYU9lbDJTZjZGCjFLVzNXK01keTZMTWs4L1BGaEswdVpkN0trL0ZMLzRVWERPZUtlMWVrYW5YSGUyNmc4U3NTUHJuU1RDSThMMk1tQUFBQUMKRnliMjkwUUc5amNEUXRZbUZ6ZEdsdmJpNWhhVzh1WlhoaGJYQnNaUzVqYjIwQgotLS0tLUVORCBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0K
  ssh-publickey: ZWNkc2Etc2hhMi1uaXN0cDUyMSBBQUFBRTJWalpITmhMWE5vWVRJdGJtbHpkSEExTWpFQUFBQUlibWx6ZEhBMU1qRUFBQUNGQkFCbk84aVFQSG9NSHFnUHlxUFUwc1ZVV2dzTU5qNVVZNTZSTWFNL0pXYmRiZ2p5blh0TkRrT2dIN0pWZzA3R3NDbGp6N1dKS0VQWFN6T3BkS0NBdkJTZGlRQkZ0Y2cxUlBYOGpLUzlvZmNya3hXY3lWcFk4QTZXU0pMaXVYTXdaK2FuUXgxb2JjcXZ4R3JOcmExcnlTaUR3RGgyOVQ0ZlorOE9aVHhnemRyczMxUlZFQT09IHJvb3RAb2NwNC1iYXN0aW9uLmFpby5leGFtcGxlLmNvbQo=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"ssh-privatekey":"LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFyQUFBQUJObFkyUnpZUwoxemFHRXlMVzVwYzNSd05USXhBQUFBQ0c1cGMzUndOVEl4QUFBQWhRUUFaenZJa0R4NkRCNm9EOHFqMU5MRlZGb0xERFkrClZHT2VrVEdqUHlWbTNXNEk4cDE3VFE1RG9CK3lWWU5PeHJBcFk4KzFpU2hEMTBzenFYU2dnTHdVbllrQVJiWElOVVQxL0kKeWt2YUgzSzVNVm5NbGFXUEFPbGtpUzRybHpNR2ZtcDBNZGFHM0tyOFJxemEydGE4a29nOEE0ZHZVK0gyZnZEbVU4WU0zYQo3TjlVVlJBQUFBRWdQa1ZtSFQ1RlpoMEFBQUFUWldOa2MyRXRjMmhoTWkxdWFYTjBjRFV5TVFBQUFBaHVhWE4wY0RVeU1RCkFBQUlVRUFHYzd5SkE4ZWd3ZXFBL0tvOVRTeFZSYUN3dzJQbFJqbnBFeG96OGxadDF1Q1BLZGUwME9RNkFmc2xXRFRzYXcKS1dQUHRZa29ROWRMTTZsMG9JQzhGSjJKQUVXMXlEVkU5ZnlNcEwyaDl5dVRGWnpKV2xqd0RwWklrdUs1Y3pCbjVxZERIVwpodHlxL0VhczJ0cld2SktJUEFPSGIxUGg5bjd3NWxQR0ROMnV6ZlZGVVFBQUFBUWdFeVNPRHY2R0J0Uk1nYU9lbDJTZjZGCjFLVzNXK01keTZMTWs4L1BGaEswdVpkN0trL0ZMLzRVWERPZUtlMWVrYW5YSGUyNmc4U3NTUHJuU1RDSThMMk1tQUFBQUMKRnliMjkwUUc5amNEUXRZbUZ6ZEdsdmJpNWhhVzh1WlhoaGJYQnNaUzVqYjIwQgotLS0tLUVORCBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0K","ssh-publickey":"ZWNkc2Etc2hhMi1uaXN0cDUyMSBBQUFBRTJWalpITmhMWE5vWVRJdGJtbHpkSEExTWpFQUFBQUlibWx6ZEhBMU1qRUFBQUNGQkFCbk84aVFQSG9NSHFnUHlxUFUwc1ZVV2dzTU5qNVVZNTZSTWFNL0pXYmRiZ2p5blh0TkRrT2dIN0pWZzA3R3NDbGp6N1dKS0VQWFN6T3BkS0NBdkJTZGlRQkZ0Y2cxUlBYOGpLUzlvZmNya3hXY3lWcFk4QTZXU0pMaXVYTXdaK2FuUXgxb2JjcXZ4R3JOcmExcnlTaUR3RGgyOVQ0ZlorOE9aVHhnemRyczMxUlZFQT09IHJvb3RAb2NwNC1iYXN0aW9uLmFpby5leGFtcGxlLmNvbQo="},"kind":"Secret","metadata":{"annotations":{},"creationTimestamp":"2024-06-27T14:36:05Z","name":"nova-migration-ssh-key","namespace":"openstack","resourceVersion":"854561","uid":"52f47d82-60b4-415d-b3b9-4f64575f442d"},"type":"Opaque"}
  name: nova-migration-ssh-key
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "0"
type: Opaque
```

### 3.4 컨트롤-플레인 서비스를 위한 시크릿 생성

~/osp-ng-ctlplane-secret.yaml
```yaml
apiVersion: v1
data:
  AdminPassword: b3BlbnN0YWNr
  AodhPassword: b3BlbnN0YWNr
  AodhDatabasePassword: b3BlbnN0YWNr
  BarbicanDatabasePassword: b3BlbnN0YWNr
  BarbicanPassword: b3BlbnN0YWNr
  BarbicanSimpleCryptoKEK: b3BlbnN0YWNr
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
kind: Secret
metadata:
  name: osp-secret
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "0"
type: Opaque
```

### 3.5 컨트롤-플레인 서비스 구ㅓ

~/osp-ng-ctlplane-deploy.yaml
```yaml
apiVersion: core.openstack.org/v1beta1
kind: OpenStackControlPlane
metadata:
  name: openstack-galera-network-isolation
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
              spec:
                type: LoadBalancer
      cinderScheduler:
        replicas: 1
      cinderBackup:
        networkAttachments:
        - storage
        replicas: 0 # backend needs to be configured
      cinderVolumes:
        nfs:
          networkAttachments:
          - storage
          customServiceConfig: |
            [nfs]
            volume_backend_name=nfs
            volume_driver=cinder.volume.drivers.nfs.NfsDriver
            nfs_snapshot_support=true
            nas_secure_file_operations=false
            nas_secure_file_permissions=false
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
              spec:
                type: LoadBalancer
      barbicanWorker:
        replicas: 1
      barbicanKeystoneListener:
        replicas: 1
  extraMounts:
  - extraVol:
    - extraVolType: Nfs
      mounts:
      - mountPath: /var/lib/glance/images
        name: nfs
      propagation:
      - Glance
      volumes:
      - name: nfs
        nfs:
          path: /nfs/glance
          server: 192.168.123.100
    name: r1
    region: r1
  glance:
    enabled: true
    template:
      databaseInstance: openstack
      customServiceConfig: |
         [DEFAULT]
         enabled_backends = default_backend:file
         [glance_store]
         default_backend = default_backend
         [default_backend]
         filesystem_store_datadir = /var/lib/glance/images/
      storage:
        storageRequest: 10G
        storageClass: "ocs-storagecluster-ceph-rbd"
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
                    metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
              spec:
                type: LoadBalancer
      metadataServiceTemplate:
        override:
          service:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.89
            spec:
              type: LoadBalancer
      schedulerServiceTemplate:
        override:
          service:
            metadata:
              annotations:
                metallb.universe.tf/address-pool: internalapi
                metallb.universe.tf/allow-shared-ip: internalapi
                metallb.universe.tf/loadBalancerIPs: 172.17.0.89
            spec:
              type: LoadBalancer
      cellTemplates:
        cell0:
          cellDatabaseAccount: nova-cell0
          cellDatabaseInstance: openstack
          cellMessageBusInstance: rabbitmq
          hasAPIAccess: true
        cell1:
          cellDatabaseAccount: nova-cell1
          cellDatabaseInstance: openstack-cell1
          cellMessageBusInstance: rabbitmq-cell1
          noVNCProxyServiceTemplate:
            enabled: true
            networkAttachments:
            - internalapi
            - ctlplane
          hasAPIAccess: true
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
          passwordSelectors:
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
        ipaddr: 172.17.0.89
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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
                  metallb.universe.tf/loadBalancerIPs: 172.17.0.89
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

## 4. 컨트롤-플레인 서비스 배포 및 확인

### 4.1 컨트롤-플레인 서비스 배포를 위한 ArgoCD 앱 생성

ArgoCD 앱 YAML
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openstack-deployment-control-plane
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/openstack-control-plane-deployment
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

ArgoCD 앱 생성
```bash
cat << EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openstack-deployment-control-plane
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/openstack-control-plane-deployment
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
* GitOps 콘솔에 로그인 하여 RHOSO 배포 확인
* 오픈스택 컨트롤-플레인은 *openstackcontrolplane* 커스텀 리소스에 의해 관리됨

### 4.2 오픈스택 컨트롤-플레인 확인

# 4.2.1 OpenStackControlPlane CRD 정의 및 스펙 스키마 확인

```bash
oc describe crd openstackcontrolplane | more
oc explain openstackcontrolplane.spec
```

실행 결과
```

```

#### 4.2.2 OpenStackControlPlane 리소스 설치 완료 확인

```bash
oc get openstackcontrolplane -n openstack
```

실행 결과
```

```
* *MESSAGE* 필드의 값이 *Setup compleate*인 것을 확인

#### 4.2.3 오픈스택 네임스페이스 상의ㅔ 포드 확인

```bash
oc get pods -n openstack
```

실행 결과
```

```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 네트워크 구성 <<](./prepare-openshfit-for-rhoso-network-isolation-via-argocd.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 데이터-플레인 구성 >>](./configure-data-plane-via-argocd.md)