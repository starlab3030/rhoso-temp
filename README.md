# 오픈스택 서비스

Red Hat OpenStack Service on OpenShift (RHOSO) 설치
* 기본적인 방식으로 오픈시프트 상에서 오픈스택 설치
  - 오픈스택 서비스 설치 사전 준비, 설치까지 오픈시프트가 제공하는 CLI(oc 명령어) 및 GUI를 통해 설치
  - 오프스택을 설치하는 기본적인 방법
  
* ArgoCD를 통한 GitOps 형태로 오픈스택 설치
  - 오프스택 설치에 필요한 리소스를 GitHub에 YAML로 구성 후, ArgoCD 앱을 생성하여 설치
  - 오픈시프트가 제공하는 GitOps를 통해 빠르고 쉽게 설치 가능


<img align="left" src="/common-images/이승일--II_컴퓨터.png" width="280px" height="280px" title="100px" alt="안녕"></img>

<br>
<br>
오픈스택 설치
<br>
<br>

&nbsp;&nbsp;1. [사전 준비](beta-lab/pre-requisite-ops.md)<br>
&nbsp;&nbsp;2. [오픈스택 서비스 오퍼레이터 설치](beta-lab/install-oso-operators.md)<br>
&nbsp;&nbsp;3. [오픈스택 서비스 보안 접근 구성](beta-lab/provide-secure-access-to-rhoso.md)<br>
&nbsp;&nbsp;4. [오픈스택 서비스 네트워크 구성](beta-lab/prepare-openshfit-for-rhoso-network-isolation.md)<br>
&nbsp;&nbsp;5. [오픈스택 서비스 컨트롤-플레인 생성](beta-lab/create-oso-ctl-plane.md)<br>
&nbsp;&nbsp;6. [오픈스택 데이터-플레인 구성](beta-lab/configure-data-plane.md)<br>
&nbsp;&nbsp;7. [오픈스택 접속](beta-lab/access-openstack.md)<br>
&nbsp;&nbsp;8. [오픈스택 노드 확장](beta-lab/scale-out-compute.md)<br>
<br>

<hr>

<img align="left" src="/common-images/이승일--II_그래서.png" width="270px" height="270px" title="100px" alt="안녕"></img>

<br>
<br>
ArgoCD를 기반으로 오픈스택 설치
<br>
<br>

&nbsp;&nbsp;1. [ArgoCD 설치](beta-lab-via-argocd/install-argocd.md)<br>
&nbsp;&nbsp;2. [사전 준비](beta-lab-via-argocd/pre-requisite-ops-via-argocd.md)<br>
&nbsp;&nbsp;3. [오픈스택 서비스 오퍼레이터 설치](beta-lab-via-argocd/install-oso-operators-via-argocd.md)<br>
&nbsp;&nbsp;4. [오프스택 서비스 네트워크 구성](beta-lab-via-argocd/prepare-openshfit-for-rhoso-network-isolation-via-argocd.md)<br>
&nbsp;&nbsp;5. [오픈스택 서비스 컨트롤-플레인 생성](beta-lab-via-argocd/create-oso-ctl-plane-via-argocd.md)<br>
&nbsp;&nbsp;6. [오픈스택 데이터-플레인 구성](beta-lab-via-argocd/configure-data-plane-via-argocd.md)<br>
&nbsp;&nbsp;7. [오픈스택 노드 확장](beta-lab-via-argocd/scale-out-compute.md)<br>
<br>
