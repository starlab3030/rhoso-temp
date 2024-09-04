# 오픈스택 접근

## 1. 오픈스택 서비스 확인

```bash
oc get pod --field-selector=status.phase=Running -n openstack --sort-by=metadata.name
```

실행 결과
```
[root@ocp4-bastion ~]# oc get pod --field-selector=status.phase=Running -n openstack --sort-by=metadata.name
NAME                           READY   STATUS    RESTARTS        AGE
ceilometer-0                   4/4     Running   0               6h53m
cinder-api-0                   2/2     Running   0               6h55m
cinder-scheduler-0             2/2     Running   0               6h55m
cinder-volume-nfs-0            2/2     Running   0               6h55m
dnsmasq-dns-5bddd8745f-gmdz9   1/1     Running   0               70m
glance-default-single-0        3/3     Running   0               6h55m
keystone-59f66f676f-6dtqp      1/1     Running   0               6h55m
memcached-0                    1/1     Running   0               6h57m
neutron-768c88f7c4-g4ccq       2/2     Running   0               6h55m
nova-api-0                     2/2     Running   0               6h53m
nova-cell0-conductor-0         1/1     Running   0               6h54m
nova-cell1-conductor-0         1/1     Running   0               6h53m
nova-cell1-novncproxy-0        1/1     Running   0               6h54m
nova-metadata-0                2/2     Running   0               6h53m
nova-scheduler-0               1/1     Running   0               6h53m
openstack-cell1-galera-0       1/1     Running   0               6h57m
openstack-galera-0             1/1     Running   0               6h57m
openstackclient                1/1     Running   0               6h55m
ovn-controller-d5x4s           1/1     Running   0               6h57m
ovn-controller-ovs-bx9jw       2/2     Running   0               6h57m
ovn-controller-ovs-scnw6       2/2     Running   0               6h57m
ovn-controller-ovs-sw478       2/2     Running   0               6h57m
ovn-controller-ttxb5           1/1     Running   1 (6h57m ago)   6h57m
ovn-controller-zh7ph           1/1     Running   0               6h57m
ovn-northd-5c4ccbb46d-pmlns    1/1     Running   0               6h56m
ovsdbserver-nb-0               1/1     Running   0               6h57m
ovsdbserver-sb-0               1/1     Running   0               6h57m
placement-c698f6c9f-hlpgd      2/2     Running   0               6h55m
rabbitmq-cell1-server-0        1/1     Running   0               6h57m
rabbitmq-server-0              1/1     Running   0               6h57m

[root@ocp4-bastion ~]#
```
<br>

## 2. 오픈스택 클라이언트

### 2.1 오픈스택 클라이언트 확인

```bash
oc get pod/openstackclient -n openstack
oc get pod/openstackclient -n openstack -o json | jq -r '.metadata.annotations."k8s.ovn.org/pod-networks"' | jq '.'
```

실행 결과
```
[root@ocp4-bastion ~]# oc get pod/openstackclient -n openstack
NAME              READY   STATUS    RESTARTS   AGE
openstackclient   1/1     Running   0          7h4m

[root@ocp4-bastion ~]# oc get pod/openstackclient -n openstack -o json | jq -r '.metadata.annotations."k8s.ovn.org/pod-networks"' | jq '.'
{
  "default": {
    "ip_addresses": [
      "10.129.2.58/23"
    ],
    "mac_address": "0a:58:0a:81:02:3a",
    "gateway_ips": [
      "10.129.2.1"
    ],
    "routes": [
      {
        "dest": "10.128.0.0/14",
        "nextHop": "10.129.2.1"
      },
      {
        "dest": "172.30.0.0/16",
        "nextHop": "10.129.2.1"
      },
      {
        "dest": "100.64.0.0/16",
        "nextHop": "10.129.2.1"
      }
    ],
    "ip_address": "10.129.2.58/23",
    "gateway_ip": "10.129.2.1"
  }
}

[root@ocp4-bastion ~]#
```

### 2.2 오픈스택 클라이언트에 접속

```bash
oc -n openstack rsh openstackclient
cd /home/cloud-admin
ls -lh .config/openstack
cat .config/openstack/secure.yaml
cat .config/openstack/clouds.yaml
```

실행 결과
```
[root@ocp4-bastion ~]# oc -n openstack rsh openstackclient

sh-5.1$ cd /home/cloud-admin

sh-5.1$ ls -lh .config/openstack
total 8.0K
-rw-r--r--. 1 root root 298 Aug 30 00:09 clouds.yaml
-rw-r--r--. 1 root root  67 Aug 30 00:09 secure.yaml

sh-5.1$ cat .config/openstack/secure.yaml
clouds:
    default:
        auth:
            password: openstack

sh-5.1$ cat .config/openstack/clouds.yaml
clouds:
    default:
        auth:
            auth_url: http://keystone-public-openstack.apps.wcv2t.dynamic.redhatworkshops.io
            project_name: admin
            username: admin
            user_domain_name: Default
            project_domain_name: Default
        region_name: regionOne

sh-5.1$
```

### 2.3 오픈스택 서비스 확인

#### 2.3.1 컴퓨트 서비스

```bash
openstack compute service list
```

실행 결과
```
sh-5.1$ openstack compute service list
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host                                    | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+
| a1fe4d67-acc3-459c-9e47-1a642b844fe2 | nova-conductor | nova-cell0-conductor-0                  | internal | enabled | up    | 2024-08-30T07:29:47.000000 |
| 3e50fc7d-f358-4e9e-ac1c-69b07efd8f52 | nova-scheduler | nova-scheduler-0                        | internal | enabled | up    | 2024-08-30T07:29:47.000000 |
| 7109dd83-3d66-4107-a1d7-baf7653cf6e1 | nova-conductor | nova-cell1-conductor-0                  | internal | enabled | up    | 2024-08-30T07:29:46.000000 |
| 124b4605-906f-4e1b-8c7c-5ec4318fa8f7 | nova-compute   | edpm-compute-0.ctlplane.aio.example.com | nova     | enabled | up    | 2024-08-30T07:29:45.000000 |
+--------------------------------------+----------------+-----------------------------------------+----------+---------+-------+----------------------------+
sh-5.1$
```

#### 2.3.2 네트워크 에이전트 리스트

```bash
openstack network agent list
```

실행 결과
```
sh-5.1$ openstack network agent list
+--------------------------------------+------------------------------+-----------------------------------------+-------------------+-------+-------+----------------+
| ID                                   | Agent Type                   | Host                                    | Availability Zone | Alive | State | Binary         |
+--------------------------------------+------------------------------+-----------------------------------------+-------------------+-------+-------+----------------+
| 32b05af9-1ccf-4e7d-9102-1fcad54fd85a | OVN Controller Gateway agent | ocp4-worker1.aio.example.com            |                   | :-)   | UP    | ovn-controller |
| d3829930-2851-4dc1-a7f5-afd2bb5a6b06 | OVN Controller Gateway agent | ocp4-worker2.aio.example.com            |                   | :-)   | UP    | ovn-controller |
| 8adf891a-9cc3-4e5d-a7ce-7ba4933b6755 | OVN Controller Gateway agent | ocp4-worker3.aio.example.com            |                   | :-)   | UP    | ovn-controller |
| 61778d26-e5e8-4947-b025-3ad4f923b51a | OVN Controller agent         | edpm-compute-0.ctlplane.aio.example.com |                   | :-)   | UP    | ovn-controller |
+--------------------------------------+------------------------------+-----------------------------------------+-------------------+-------+-------+----------------+
sh-5.1$
```
<br>

## 3. 가상머신 생성

### 3.1 컴퓨트 노드를 Cell에 매핑

```bash
oc rsh nova-cell0-conductor-0 nova-manage cell_v2 discover_hosts --verbose
```

실행 결과
```
[root@ocp4-bastion ~]# oc rsh nova-cell0-conductor-0 nova-manage cell_v2 discover_hosts --verbose
Modules with known eventlet monkey patching issues were imported prior to eventlet monkey patching: urllib3. This warning can usually be ignored if the caller is only importing and not executing nova code.
2024-08-30 07:34:58.605 85151 WARNING oslo_policy.policy [None req-169b18ad-f4be-413a-a6cd-0d90a8cb9438 - - - - - -] JSON formatted policy_file support is deprecated since Victoria release. You need to use YAML format which will be default in future. You can use ``oslopolicy-convert-json-to-yaml`` tool to convert existing JSON-formatted policy file to YAML-formatted in backward compatible way: https://docs.openstack.org/oslo.policy/latest/cli/oslopolicy-convert-json-to-yaml.html.
2024-08-30 07:34:58.605 85151 WARNING oslo_policy.policy [None req-169b18ad-f4be-413a-a6cd-0d90a8cb9438 - - - - - -] JSON formatted policy_file support is deprecated since Victoria release. You need to use YAML format which will be default in future. You can use ``oslopolicy-convert-json-to-yaml`` tool to convert existing JSON-formatted policy file to YAML-formatted in backward compatible way: https://docs.openstack.org/oslo.policy/latest/cli/oslopolicy-convert-json-to-yaml.html.
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 46031296-eae2-4d8c-ac02-47e546f42325
Checking host mapping for compute host 'edpm-compute-0.ctlplane.aio.example.com': 3f5c43d7-f332-4858-89f3-aee4bd8df2b0
Creating host mapping for compute host 'edpm-compute-0.ctlplane.aio.example.com': 3f5c43d7-f332-4858-89f3-aee4bd8df2b0
Found 1 unmapped computes in cell: 46031296-eae2-4d8c-ac02-47e546f42325

[root@ocp4-bastion ~]#
```

### 3.2 오픈스택 클라이언트 준비

```bash
oc rsh -n openstack openstackclient
pwd
export GATEWAY=192.168.123.1
export PUBLIC_NETWORK_CIDR=192.168.123.1/24
export PRIVATE_NETWORK_CIDR=192.168.100.0/24
export PUBLIC_NET_START=192.168.123.91
export PUBLIC_NET_END=192.168.123.99
export DNS_SERVER=8.8.8.8
```

실행 결과
```

[root@ocp4-bastion ~]# oc rsh -n openstack openstackclient

sh-5.1$ pwd
/home/cloud-admin

sh-5.1$ export GATEWAY=192.168.123.1

sh-5.1$ export PUBLIC_NETWORK_CIDR=192.168.123.1/24

sh-5.1$ export PRIVATE_NETWORK_CIDR=192.168.100.0/24

sh-5.1$ export PUBLIC_NET_START=192.168.123.91

sh-5.1$ export PUBLIC_NET_END=192.168.123.99

sh-5.1$ export DNS_SERVER=8.8.8.8

sh-5.1$
```

### 3.3 플레이버 생성

```bash
openstack flavor create --ram 512 --disk 1 --vcpu 1 --public tiny
openstack flavor list
```

실행 결과
```
sh-5.1$ openstack flavor create --ram 512 --disk 1 --vcpu 1 --public tiny
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| description                | None                                 |
| disk                       | 1                                    |
| id                         | dd9c7e23-fb0b-41c8-9179-426aaf2de6e2 |
| name                       | tiny                                 |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 512                                  |
| rxtx_factor                | 1.0                                  |
| swap                       |                                      |
| vcpus                      | 1                                    |
+----------------------------+--------------------------------------+

sh-5.1$ openstack flavor list
+--------------------------------------+------+-----+------+-----------+-------+-----------+
| ID                                   | Name | RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+------+-----+------+-----------+-------+-----------+
| dd9c7e23-fb0b-41c8-9179-426aaf2de6e2 | tiny | 512 |    1 |         0 |     1 | True      |
+--------------------------------------+------+-----+------+-----------+-------+-----------+

sh-5.1$
```

### 3.4 이미지 생성

```bash
curl -O -L https://github.com/cirros-dev/cirros/releases/download/0.6.2/cirros-0.6.2-x86_64-disk.img
openstack image create cirros --container-format bare --disk-format qcow2 --public --file cirros-0.6.2-x86_64-disk.img
openstack image list
```

실행 결과
```
sh-5.1$ curl -O -L https://github.com/cirros-dev/cirros/releases/download/0.6.2/cirros-0.6.2-x86_64-disk.img
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 20.4M  100 20.4M    0     0  18.8M      0  0:00:01  0:00:01 --:--:--  214M

sh-5.1$ openstack image create cirros --container-format bare --disk-format qcow2 --public --file cirros-0.6.2-x86_64-disk.img
+------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                      |
+------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+
| container_format | bare                                                                                                                                                       |
| created_at       | 2024-08-30T08:13:17Z                                                                                                                                       |
| disk_format      | qcow2                                                                                                                                                      |
| file             | /v2/images/ceeffbb7-0b5b-47d1-926e-c27f308d52c8/file                                                                                                       |
| id               | ceeffbb7-0b5b-47d1-926e-c27f308d52c8                                                                                                                       |
| min_disk         | 0                                                                                                                                                          |
| min_ram          | 0                                                                                                                                                          |
| name             | cirros                                                                                                                                                     |
| owner            | be11b65a769146c5942a52540f24face                                                                                                                           |
| properties       | locations='[]', os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/cirros', owner_specified.openstack.sha256='' |
| protected        | False                                                                                                                                                      |
| schema           | /v2/schemas/image                                                                                                                                          |
| status           | queued                                                                                                                                                     |
| tags             |                                                                                                                                                            |
| updated_at       | 2024-08-30T08:13:17Z                                                                                                                                       |
| visibility       | public                                                                                                                                                     |
+------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------+

sh-5.1$ openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| ceeffbb7-0b5b-47d1-926e-c27f308d52c8 | cirros | active |
+--------------------------------------+--------+--------+

sh-5.1$
```

### 3.5 키-페어 생성

```bash
ssh-keygen -m PEM -t rsa -b 2048 -f ~/.ssh/id_rsa_pem
openstack keypair create --public-key ~/.ssh/id_rsa_pem.pub default
openstack keypair list
```

실행 결과
```
sh-5.1$ ssh-keygen -m PEM -t rsa -b 2048 -f ~/.ssh/id_rsa_pem
Generating public/private rsa key pair.
Created directory '/home/cloud-admin/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/cloud-admin/.ssh/id_rsa_pem
Your public key has been saved in /home/cloud-admin/.ssh/id_rsa_pem.pub
The key fingerprint is:
SHA256:mpTK0HlKqVQ3gTiQcRBAvzcrcAwBKgvUIpx8/28q3yI cloud-admin@openstackclient
The key's randomart image is:
+---[RSA 2048]----+
|&X+. ..          |
|=*=o.  .         |
|=.ooo o          |
|o.oo.= o         |
|..o+=o= S        |
| .o=.=o+         |
|  ..+.o .        |
|    .E ..o       |
|      +o+.       |
+----[SHA256]-----+

sh-5.1$ openstack keypair create --public-key ~/.ssh/id_rsa_pem.pub default
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| created_at  | None                                            |
| fingerprint | 15:06:3f:73:b0:94:a3:cd:5d:ef:89:ba:2d:51:f2:49 |
| id          | default                                         |
| is_deleted  | None                                            |
| name        | default                                         |
| type        | ssh                                             |
| user_id     | 22a56e209929479fa1f4b7dc5a337636                |
+-------------+-------------------------------------------------+

sh-5.1$ openstack keypair list
+---------+-------------------------------------------------+------+
| Name    | Fingerprint                                     | Type |
+---------+-------------------------------------------------+------+
| default | 15:06:3f:73:b0:94:a3:cd:5d:ef:89:ba:2d:51:f2:49 | ssh  |
+---------+-------------------------------------------------+------+

sh-5.1$
```

### 3.6 보안 그룹 생성

```bash
openstack security group create basic
openstack security group list
openstack security group rule create basic --protocol tcp --dst-port 22:22 --remote-ip 0.0.0.0/0
openstack security group rule create --protocol icmp basic
openstack security group rule create --protocol udp --dst-port 53:53 basic
openstack security group rule list basic
```

실행 결과
```
sh-5.1$ openstack security group create basic
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field           | Value                                                                                                                                                                        |
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at      | 2024-08-30T08:23:04Z                                                                                                                                                         |
| description     | basic                                                                                                                                                                        |
| id              | 0f984317-51d8-4695-906a-e20880610ba6                                                                                                                                         |
| name            | basic                                                                                                                                                                        |
| project_id      | be11b65a769146c5942a52540f24face                                                                                                                                             |
| revision_number | 1                                                                                                                                                                            |
| rules           | created_at='2024-08-30T08:23:04Z', direction='egress', ethertype='IPv6', id='b4c0b13d-1431-4982-81b7-083019269777', standard_attr_id='12', updated_at='2024-08-30T08:23:04Z' |
|                 | created_at='2024-08-30T08:23:04Z', direction='egress', ethertype='IPv4', id='ec14044d-d554-4b6f-b964-33240974e8a8', standard_attr_id='11', updated_at='2024-08-30T08:23:04Z' |
| shared          | False                                                                                                                                                                        |
| stateful        | True                                                                                                                                                                         |
| tags            | []                                                                                                                                                                           |
| updated_at      | 2024-08-30T08:23:04Z                                                                                                                                                         |
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

sh-5.1$ openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+------+
| ID                                   | Name    | Description            | Project                          | Tags |
+--------------------------------------+---------+------------------------+----------------------------------+------+
| 043ec037-88a4-4811-a322-12fee090fa23 | default | Default security group | be11b65a769146c5942a52540f24face | []   |
| 0f984317-51d8-4695-906a-e20880610ba6 | basic   | basic                  | be11b65a769146c5942a52540f24face | []   |
+--------------------------------------+---------+------------------------+----------------------------------+------+

sh-5.1$ openstack security group rule create basic --protocol tcp --dst-port 22:22 --remote-ip 0.0.0.0/0
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-08-30T08:24:09Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | eb7ba55f-f0a9-4aef-8d8e-d3b8a7ea5184 |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | 22                                   |
| port_range_min          | 22                                   |
| project_id              | be11b65a769146c5942a52540f24face     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 0f984317-51d8-4695-906a-e20880610ba6 |
| tags                    | []                                   |
| updated_at              | 2024-08-30T08:24:09Z                 |
+-------------------------+--------------------------------------+

sh-5.1$ openstack security group rule create basic --protocol icmp
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-08-30T08:25:36Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 5afa0e50-ca64-4b78-9f7b-5f8e7b85031b |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | None                                 |
| port_range_min          | None                                 |
| project_id              | be11b65a769146c5942a52540f24face     |
| protocol                | icmp                                 |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 0f984317-51d8-4695-906a-e20880610ba6 |
| tags                    | []                                   |
| updated_at              | 2024-08-30T08:25:36Z                 |
+-------------------------+--------------------------------------+

sh-5.1$ openstack security group rule create basic --protocol udp --dst-port 53:53
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2024-08-30T08:25:51Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 54b24d00-a754-43aa-877a-982bae1fdd8d |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | 53                                   |
| port_range_min          | 53                                   |
| project_id              | be11b65a769146c5942a52540f24face     |
| protocol                | udp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 0f984317-51d8-4695-906a-e20880610ba6 |
| tags                    | []                                   |
| updated_at              | 2024-08-30T08:25:51Z                 |
+-------------------------+--------------------------------------+

sh-5.1$ openstack security group rule list basic
+--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
| ID                                   | IP Protocol | Ethertype | IP Range  | Port Range | Direction | Remote Security Group | Remote Address Group |
+--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+
| 54b24d00-a754-43aa-877a-982bae1fdd8d | udp         | IPv4      | 0.0.0.0/0 | 53:53      | ingress   | None                  | None                 |
| 5afa0e50-ca64-4b78-9f7b-5f8e7b85031b | icmp        | IPv4      | 0.0.0.0/0 |            | ingress   | None                  | None                 |
| b4c0b13d-1431-4982-81b7-083019269777 | None        | IPv6      | ::/0      |            | egress    | None                  | None                 |
| eb7ba55f-f0a9-4aef-8d8e-d3b8a7ea5184 | tcp         | IPv4      | 0.0.0.0/0 | 22:22      | ingress   | None                  | None                 |
| ec14044d-d554-4b6f-b964-33240974e8a8 | None        | IPv4      | 0.0.0.0/0 |            | egress    | None                  | None                 |
+--------------------------------------+-------------+-----------+-----------+------------+-----------+-----------------------+----------------------+

sh-5.1$
```

### 3.7 네트워크 생성

#### 3.7.1 퍼블릭 네트워크 생성

```bash
openstack network create --external --provider-physical-network datacentre --provider-network-type flat public
openstack subnet create public-net \
--subnet-range $PUBLIC_NETWORK_CIDR \
--no-dhcp \
--gateway $GATEWAY \
--allocation-pool start=$PUBLIC_NET_START,end=$PUBLIC_NET_END \
--network public
```

실행 결과
```
sh-5.1$ openstack network create --external --provider-physical-network datacentre --provider-network-type flat public
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-08-30T08:30:26Z                 |
| description               |                                      |
| dns_domain                |                                      |
| id                        | 23890c56-be26-47e9-80c5-28e936191dd8 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| l2_adjacency              | True                                 |
| mtu                       | 1500                                 |
| name                      | public                               |
| port_security_enabled     | True                                 |
| project_id                | be11b65a769146c5942a52540f24face     |
| provider:network_type     | flat                                 |
| provider:physical_network | datacentre                           |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | be11b65a769146c5942a52540f24face     |
| updated_at                | 2024-08-30T08:30:26Z                 |
+---------------------------+--------------------------------------+

sh-5.1$ openstack subnet create public-net \
--subnet-range $PUBLIC_NETWORK_CIDR \
--no-dhcp \
--gateway $GATEWAY \
--allocation-pool start=$PUBLIC_NET_START,end=$PUBLIC_NET_END \
--network public
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.123.91-192.168.123.99        |
| cidr                 | 192.168.123.0/24                     |
| created_at           | 2024-08-30T08:30:39Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | False                                |
| gateway_ip           | 192.168.123.1                        |
| host_routes          |                                      |
| id                   | c791b0b9-7bc3-455c-bd6d-a772e3636374 |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | public-net                           |
| network_id           | 23890c56-be26-47e9-80c5-28e936191dd8 |
| project_id           | be11b65a769146c5942a52540f24face     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-08-30T08:30:39Z                 |
+----------------------+--------------------------------------+

sh-5.1$
```

#### 3.7.2 프라이빗 네트워크 생성

```bash
openstack network create --internal private
openstack subnet create private-net \
--subnet-range $PRIVATE_NETWORK_CIDR \
--network private
```

실행 결과
```
sh-5.1$ openstack network create --internal private
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-08-30T08:31:16Z                 |
| description               |                                      |
| dns_domain                |                                      |
| id                        | 42fa46cd-608b-4fa0-9ce5-a7f1f57baf29 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| l2_adjacency              | True                                 |
| mtu                       | 1442                                 |
| name                      | private                              |
| port_security_enabled     | True                                 |
| project_id                | be11b65a769146c5942a52540f24face     |
| provider:network_type     | geneve                               |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 44247                                |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| tenant_id                 | be11b65a769146c5942a52540f24face     |
| updated_at                | 2024-08-30T08:31:17Z                 |
+---------------------------+--------------------------------------+

sh-5.1$ openstack subnet create private-net \
--subnet-range $PRIVATE_NETWORK_CIDR \
--network private
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.168.100.2-192.168.100.254        |
| cidr                 | 192.168.100.0/24                     |
| created_at           | 2024-08-30T08:31:23Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.168.100.1                        |
| host_routes          |                                      |
| id                   | df3d14b4-16ff-4089-affa-585bd24960ba |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | private-net                          |
| network_id           | 42fa46cd-608b-4fa0-9ce5-a7f1f57baf29 |
| project_id           | be11b65a769146c5942a52540f24face     |
| revision_number      | 0                                    |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-08-30T08:31:23Z                 |
+----------------------+--------------------------------------+

sh-5.1$
```

#### 3.7.3 라우터 생성

```bash
openstack router create vrouter
openstack router set vrouter --external-gateway public
openstack router add subnet vrouter private-net
```

실행 결과
```
sh-5.1$ openstack router create vrouter
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2024-08-30T08:33:05Z                 |
| description             |                                      |
| enable_ndp_proxy        | None                                 |
| external_gateway_info   | null                                 |
| flavor_id               | None                                 |
| id                      | 85e1a388-9e8a-4cf9-9a93-fded8ffc3a46 |
| name                    | vrouter                              |
| project_id              | be11b65a769146c5942a52540f24face     |
| revision_number         | 1                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tags                    |                                      |
| tenant_id               | be11b65a769146c5942a52540f24face     |
| updated_at              | 2024-08-30T08:33:05Z                 |
+-------------------------+--------------------------------------+

sh-5.1$ openstack router set vrouter --external-gateway public

sh-5.1$ openstack router add subnet vrouter private-net

sh-5.1$
```


### 3.8 가상머신 생성

```bash
openstack server create \
    --flavor tiny --key-name default --network private --security-group basic \
    --image cirros test-server
openstack server list
```

실행 결과
```
sh-5.1$ openstack server create \
    --flavor tiny --key-name default --network private --security-group basic \
    --image cirros test-server
+-------------------------------------+-----------------------------------------------+
| Field                               | Value                                         |
+-------------------------------------+-----------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                        |
| OS-EXT-AZ:availability_zone         |                                               |
| OS-EXT-SRV-ATTR:host                | None                                          |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                          |
| OS-EXT-SRV-ATTR:instance_name       |                                               |
| OS-EXT-STS:power_state              | NOSTATE                                       |
| OS-EXT-STS:task_state               | scheduling                                    |
| OS-EXT-STS:vm_state                 | building                                      |
| OS-SRV-USG:launched_at              | None                                          |
| OS-SRV-USG:terminated_at            | None                                          |
| accessIPv4                          |                                               |
| accessIPv6                          |                                               |
| addresses                           |                                               |
| adminPass                           | LC96EbtHZHpM                                  |
| config_drive                        |                                               |
| created                             | 2024-08-30T08:37:16Z                          |
| flavor                              | tiny (dd9c7e23-fb0b-41c8-9179-426aaf2de6e2)   |
| hostId                              |                                               |
| id                                  | bf48fc2e-4d89-4132-9c59-f10c02dc72ef          |
| image                               | cirros (ceeffbb7-0b5b-47d1-926e-c27f308d52c8) |
| key_name                            | default                                       |
| name                                | test-server                                   |
| progress                            | 0                                             |
| project_id                          | be11b65a769146c5942a52540f24face              |
| properties                          |                                               |
| security_groups                     | name='0f984317-51d8-4695-906a-e20880610ba6'   |
| status                              | BUILD                                         |
| updated                             | 2024-08-30T08:37:17Z                          |
| user_id                             | 22a56e209929479fa1f4b7dc5a337636              |
| volumes_attached                    |                                               |
+-------------------------------------+-----------------------------------------------+

sh-5.1$ openstack server list
+--------------------------------------+-------------+--------+------------------------+--------+--------+
| ID                                   | Name        | Status | Networks               | Image  | Flavor |
+--------------------------------------+-------------+--------+------------------------+--------+--------+
| bf48fc2e-4d89-4132-9c59-f10c02dc72ef | test-server | ACTIVE | private=192.168.100.76 | cirros | tiny   |
+--------------------------------------+-------------+--------+------------------------+--------+--------+

sh-5.1$
```

### 3.9 floating IP 생성 및 할당

```bash
openstack floating ip create public
openstack server add floating ip test-server $(openstack floating ip list -c "Floating IP Address" -f value)
openstack server show test-server -c addresses
```

실행 결과
```
sh-5.1$ openstack floating ip create public
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2024-08-30T08:39:10Z                 |
| description         |                                      |
| dns_domain          |                                      |
| dns_name            |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 192.168.123.94                       |
| floating_network_id | 23890c56-be26-47e9-80c5-28e936191dd8 |
| id                  | 6a24d49b-7890-4ee9-b7c1-96f58df337fc |
| name                | 192.168.123.94                       |
| port_details        | None                                 |
| port_forwardings    | []                                   |
| port_id             | None                                 |
| project_id          | be11b65a769146c5942a52540f24face     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2024-08-30T08:39:10Z                 |
+---------------------+--------------------------------------+

sh-5.1$ openstack server add floating ip test-server $(openstack floating ip list -c "Floating IP Address" -f value)

sh-5.1$ openstack server show test-server -c addresses
+-----------+----------------------------------------+
| Field     | Value                                  |
+-----------+----------------------------------------+
| addresses | private=192.168.100.76, 192.168.123.94 |
+-----------+----------------------------------------+

sh-5.1$
```

### 3.10 가상머신 연결 테스트

```bash
ssh cirros@192.168.123.94
ip a s
```

실행 결과
```
[root@ocp4-bastion ~]# ssh cirros@192.168.123.94
The authenticity of host '192.168.123.94 (192.168.123.94)' can't be established.
ED25519 key fingerprint is SHA256:bRbdGNxZsig88sYaMV0K1w1IAS3QjzygDehQViP5XfU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.123.94' (ED25519) to the list of known hosts.
cirros@192.168.123.94's password: gocubsgo

$ ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1442 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:1d:bf:8e brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.76/24 brd 192.168.100.255 scope global dynamic noprefixroute eth0
       valid_lft 42620sec preferred_lft 37220sec
    inet6 fe80::f816:3eff:fe1d:bf8e/64 scope link
       valid_lft forever preferred_lft forever

$ exit
```
<br>

## 4. Horizon 활성황

### 4.1 컨트롤-플레인 구성 확인

```bash
oc get openstackcontrolplanes -n openstack
oc get openstackcontrolplanes/openstack-galera-network-isolation -n openstack -o json | jq '.'
```

실행 결과
```
[root@ocp4-bastion ~]# oc get openstackcontrolplanes -n openstack
NAME                                 STATUS   MESSAGE
openstack-galera-network-isolation   True     Setup complete

[root@ocp4-bastion ~]# oc get openstackcontrolplanes/openstack-galera-network-isolation -n openstack -o json | jq '.'

...<snip>...

[root@ocp4-bastion ~]#
```

### 4.2 Horizon 구성 확인

```bash
oc get openstackcontrolplanes/openstack-galera-network-isolation -n openstack -o json | jq '.spec.horizon'
```

```json
{
  "apiOverride": {
    "route": {}
  },
  "enabled": false,
  "template": {
    "customServiceConfig": "# add your customization here",
    "memcachedInstance": "memcached",
    "override": {},
    "preserveJobs": false,
    "replicas": 1,
    "resources": {},
    "secret": "osp-secret",
    "tls": {}
  }
}
```

### 4.3 Horizon 설정 변경

```bash
oc patch openstackcontrolplanes/openstack-galera-network-isolation -p='[{"op": "replace", "path": "/spec/horizon/enabled", "value": true}]' --type json
oc patch openstackcontrolplane/openstack-galera-network-isolation -p '{"spec": {"horizon": {"template": {"customServiceConfig": "USE_X_FORWARDED_HOST = False" }}}}' --type=merge
oc patch openstackcontrolplane/openstack-galera-network-isolation -p '{"spec": {"horizon": {"apiOverride": {"route": {"spec": {"tls": {"insecureEdgeTerminationPolicy": "Allow", "termination": "edge"}}}}}}}' --type=merge
oc get openstackcontrolplanes/openstack-galera-network-isolation -n openstack -o json | jq '.spec.horizon'
```

```json
{
  "apiOverride": {
    "route": {
      "spec": {
        "tls": {
          "insecureEdgeTerminationPolicy": "Allow",
          "termination": "edge"
        },
        "to": {}
      }
    }
  },
  "enabled": true,
  "template": {
    "customServiceConfig": "USE_X_FORWARDED_HOST = False",
    "memcachedInstance": "memcached",
    "override": {},
    "preserveJobs": false,
    "replicas": 1,
    "resources": {},
    "secret": "osp-secret",
    "tls": {}
  }
}
```

### 4.4 Horizon 경로 확인

```bash
ROUTE=$(oc get routes horizon  -o go-template='https://{{range .status.ingress}}{{.host}}{{end}}')
echo $ROUTE
```

실행 결과
```
[root@ocp4-bastion ~]# ROUTE=$(oc get routes horizon  -o go-template='https://{{range .status.ingress}}{{.host}}{{end}}')

[root@ocp4-bastion ~]# echo $ROUTE
https://horizon-openstack.apps.wcv2t.dynamic.redhatworkshops.io

[root@ocp4-bastion ~]#
```
* admin/openstack으로 로그인

<br>

## 마무리

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 데이터-플레인 구성 <<](./configure-data-plane.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 노드 확장 >>](./scale-out-compute.md)