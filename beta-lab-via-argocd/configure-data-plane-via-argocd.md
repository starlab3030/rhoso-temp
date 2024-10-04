# 오픈스택 데이터-플레인 구성

## 1. 데이터-플레인 배포 준비

### 1.1 데이터 플레인 노드 설정

~/osp-ng-dataplane-node-set-deploy.yaml
```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneNodeSet
metadata:
  name: openstack-edpm-ipam
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  env:
    - name: ANSIBLE_FORCE_COLOR
      value: "True"
    - name: ANSIBLE_VERBOSITY
      value: "2"
  services:
    - bootstrap
    - configure-network
    - validate-network
    - install-os
    - configure-os
    - ssh-known-hosts
    - run-os
    - reboot-os
    - install-certs
    - ovn
    - neutron-metadata
    - libvirt
    - nova
    - telemetry
  preProvisioned: true
  networkAttachments:
  - ctlplane
  nodes:
      edpm-compute-0:
        hostName: edpm-compute-0
        ansible:
          ansibleHost: 172.22.0.100
        networks:
        - name: ctlplane
          subnetName: subnet1
          defaultRoute: false
          fixedIP: 172.22.0.100
        - name: internalapi
          subnetName: subnet1
        - name: storage
          subnetName: subnet1
        - name: tenant
          subnetName: subnet1
        - name: external
          subnetName: subnet1
  nodeTemplate:
    ansibleSSHPrivateKeySecret: dataplane-ansible-ssh-private-key-secret
    ansible:
      ansibleUser: root
      ansibleVarsFrom:
      - prefix: subscription_manager_
        secretRef:
          name: subscription-manager
      ansibleVars:
         edpm_network_config_template: |
          ---
          {% set mtu_list = [ctlplane_mtu] %}
          {% for network in nodeset_networks %}
          {{ mtu_list.append(lookup('vars', networks_lower[network] ~ '_mtu')) }}
          {%- endfor %}
          {% set min_viable_mtu = mtu_list | max %}
          network_config:
          - type: ovs_bridge
            name: {{ neutron_physical_bridge_name }}
            mtu: {{ min_viable_mtu }}
            use_dhcp: false
            dns_servers: {{ ctlplane_dns_nameservers }}
            domain: {{ dns_search_domains }}
            addresses:
            - ip_netmask: {{ ctlplane_ip }}/{{ ctlplane_cidr }}
            routes: {{ ctlplane_host_routes }}
            members:
            - type: interface
              name: nic1
              mtu: {{ min_viable_mtu }}
              # force the MAC address of the bridge to this interface
              primary: true
          {% for network in nodeset_networks if network != 'external' %}
            - type: vlan
              mtu: {{ lookup('vars', networks_lower[network] ~ '_mtu') }}
              vlan_id: {{ lookup('vars', networks_lower[network] ~ '_vlan_id') }}
              addresses:
              - ip_netmask:
                  {{ lookup('vars', networks_lower[network] ~ '_ip') }}/{{ lookup('vars', networks_lower[network] ~ '_cidr') }}
              routes: {{ lookup('vars', networks_lower[network] ~ '_host_routes') }}
          {% endfor %}
          {% if 'external' in nodeset_networks %}
          - type: ovs_bridge
            name: br-ex
            dns_servers: {{ ctlplane_dns_nameservers }}
            domain: {{ dns_search_domains }}
            use_dhcp: false
            members:
            - type: interface
              name: nic2
              mtu: 1500
              primary: true
            routes:
            - ip_netmask: 0.0.0.0/0
              next_hop: {{ external_gateway_ip | default('192.168.123.1') }}
            addresses:
            - ip_netmask: {{ external_ip }}/{{ external_cidr }}
          {% endif %}
         edpm_network_config_hide_sensitive_logs: false
          #
          # These vars are for the network config templates themselves and are
          # considered EDPM network defaults (for all computes).
         ctlplane_host_routes: []
         ctlplane_subnet_cidr: 24
         dns_search_domains: aio.example.com
         ctlplane_vlan_id: 1
         ctlplane_mtu: 1500
         external_mtu: 1500
         external_vlan_id: 44
         external_cidr: '24'
         external_host_routes: []
         internalapi_mtu: 1500
         internalapi_vlan_id: 20
         internalapi_cidr: '24'
         internalapi_host_routes: []
         storage_mtu: 1500
         storage_vlan_id: 21
         storage_cidr: '24'
         storage_host_routes: []
         tenant_mtu: 1500
         tenant_vlan_id: 22
         tenant_cidr: '24'
         tenant_host_routes: []
         neutron_physical_bridge_name: br-osp
         # name of the first network interface on the compute node:
         neutron_public_interface_name: eth0
         # edpm_nodes_validation
         edpm_nodes_validation_validate_controllers_icmp: false
         edpm_nodes_validation_validate_gateway_icmp: false
         gather_facts: false
         enable_debug: false
         edpm_sshd_allowed_ranges: ['172.22.0.0/16']
         edpm_podman_buildah_login: true
         edpm_container_registry_logins:
          registry.redhat.io:
            6340056|osp-on-ocp-lb1374: "eyJhbGciOiJSUzUxMiJ9.eyJzdWIiOiI1Y2EzM2NjNGY4NWM0MmZmYTI3YmU5Y2UyMWI3M2JjMCJ9.GAxgg6Ht2oCS8zxHdwQw9kSD6RHeQOWYaDOcnQB5RElewQKvZmcNWi-YJdInJ5iXTE9r9tGVIN7fhFJL7f-hhL1PK2RVzZHD8qyfkMWcCEF5GUvp8rDX4GDrSkqjpUD44teWYkOy9Nb-3pOGzRIC7qs88uSxMz7hfil4I_HmjF4AAPIi4j3QZhp0lqrXzzf7vt6NLlizDFa2XTcPf_vQqReFu3A_5iWfy8XmLlC7QIixeVv2IE-ahRqM_UDCf5Dg3n2WpYvmP5jcSPFOLoT7sMimyeaPBna793boiX2swmeGHQ23tx1nFavCUavGv_cDRAvzVXCJ2NROTJ5unHiN7CXEbzm4Rg-65tY4D0YynTU8L6t0gYtXYYY9_wi1xNs-cShAmCMh1ySJn9nBcq4ydvH7eQnhSEvoK0bPsN_vWJCgOQBQyOdpTfRMU6piAy9H1zJ0KzsSzuKSS8fX0m9oN7narZPl34DTiEUTDeW8_SS6vJjHr_Q9O_X4mVeeQhH2ocN_4M9R6A89tmQ2jObuWm-cu1Yk-G6FSPUONhsoC_99nQnICS4mAuCWWDHxFY61hIrreVZBSH053MgfSaG2sqTb26MkxKWx-TP1sx18pb1xmo4IQEwILIbLlSPA3vafbrbQO5RQcm3UYKtYwev0vAlL5taXiTuLEyPscdzv0Sc"
         edpm_bootstrap_command: |
           subscription-manager register --username "{{ subscription_manager_username }}" --password "{{ subscription_manager_password }}"
           sudo subscription-manager release --set=9.4
           sudo subscription-manager repos --disable=*
           sudo subscription-manager repos --enable=rhel-9-for-x86_64-baseos-eus-rpms --enable=rhel-9-for-x86_64-appstream-eus-rpms --enable=rhel-9-for-x86_64-highavailability-eus-rpms --enable=fast-datapath-for-rhel-9-x86_64-rpms --enable=rhoso-18.0-for-rhel-9-x86_64-rpms --enable=rhceph-7-tools-for-rhel-9-x86_64-rpms
```

### 1.2 데이터-플레인 노드 배포

~/osp-ng-dataplane-deployment.yaml
```yaml
apiVersion: dataplane.openstack.org/v1beta1
kind: OpenStackDataPlaneDeployment
metadata:
  name: openstack-edpm-ipam
  namespace: openstack
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  nodeSets:
    - openstack-edpm-ipam
```

### 1.3 레드햇 레지스트리 로그인을 위한 서브스크립션 및 시크릿 생성

#### 1.3.1 ID & PW 준비

로그인 ID 및 PW 변환
```bash
echo -n "your_username" | base64
echo -n "your_password" | base64
```

실행 결과
```

```

#### 1.3.2 서브스크립션을 위한 시크릿 준비

```bash
oc apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: subscription-manager
  namespace: openstack
data:
  username: <base64 encoded subscription-manager username>
  password: <base64 encoded subscription-manager password>
EOF
```

실행 결과
```

```
<br>

## 2. 데이터-플레인 배포

### 2.1 오픈스택 데이터-플레인 배포를 위한 ArgoCD 앱 매니페스트 생성

```bash
cat << EOF | oc apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: openstack-deployment-data-plane
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: 'https://github.com/pnavarro/showroom_osp-on-ocp-advanced.git'
    targetRevision: HEAD
    path: content/files/manifests/openstack-data-plane-deployment
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

### 2.2 데이터-플레인 배포 확인

GitOps 콘솔에 로그인하여 배포 확인
* *openstack-deployment-data-plane* 앱을 클릭하고 데이터-플레인 배포에 접급
* *bootstrap-openstack-edpm-ipam-openstack-edpm-ipam* 잡을 클릭하고 배포 로그엔 접근
  - 모든 로그의 진척이 끝날 때까지 기다림
* *OpenStackDataPlaneDeployment* CRD를 사용하여 데이터-플레인 노드 상의 서비스를 구성하고 데이터-플레인을 배포

#### 2.2.1 CLI에서 데이터-플레인 배포 확인

```bash
oc get openstackdataplanedeployment
```

실행 결과
```

```
* *MESSAGE* 필드가 *Setup Complete*인 것을 확인

#### 2.2.2 *OpenStackDataPlaneNodeSet* CRD 리소스 확인

```bash
oc get openstackdataplanenodeset
```
* 해당 리소스는 노드 고유 구성을 가지고 일반적인 컴퓨트 노드의 셋을 생성

실행 결과
```

```
* *MESSAGE* 필들가 *NodeSet Ready*인 것을 확인

예제에서는 사전에 프로비저닝 된 컴퓨트 *edpm-compute-0*가 openstack-edpm-ipam **OpenStackDataPlaneNodeSet** CRD에 정의됨
```bash
oc describe openstackdataplanenodeset openstack-edpm-ipam -n openstack
```

실행 결과
```

```
<br>

## 요약

<br>
<br>

------
[차례](../README.md) &nbsp;&nbsp;&nbsp;&nbsp; [<< 오픈스택 서비스 컨트롤-플레인 생성 <<](./create-oso-ctl-plane-via-argocd.md) &nbsp;&nbsp;&nbsp;&nbsp; [>> 오픈스택 노드 확장 >>](./scale-out-compute.md)