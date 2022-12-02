# Calico 기초

#### Calico이란?

calico는 컨테이너, 가상 머신 및 기본 호스트 기반 워크로드를 위한 오픈 소스 네트워킹 및 네트워크 보안 솔루션이다. calico는 Kubernetes, OpenShift, Mirantis Kubernetes Engine(MKE), OpenStack 및 베어메탈 서비스를 포함한 광범위한 플랫폼을 지원한다.

#### 구성 요소 아키텍처
![image](https://user-images.githubusercontent.com/86287920/205193350-5e35ebce-7962-4a29-83a5-b695755557b1.png)

여러가지 구성요소 많지만, 일단 눈여겨 볼 내용은 calico가 사용하는 Datastore와 Master Node를 포함한 모든 Node들에 존재하는 Calico Pods
- Felix: 인터페이스 관리, 라우팅 정보 관리, ACL 관리, 상태 체크
- Bird: BGP Peer에 라우팅 정보 전파 및 수신, BGP RR(Route Reflector
- Confd: calico global 설정과 BGP 설정 변경 시(Trigger) Bird에 적용해줌
- Datastore plugin: calico 설정 정보를 저장하는 곳 - k8s API datastore(kdd) 혹은 etcd 중 선택
- Calico IPAM plugin: Cluster 내에서 파드에 할당할 IP 대역
- calico-kube-controllers: calico 동작 관련 감시(watch
- calicoctl: calico 오븢ㄱ트를 CRUD 할 수 있다. 즉 Datastore 접근 가능

#### BGP(Border Gateway Protocol)란?

AS 사이에서 이용되는 라우팅 프로토콜이다. 대규모 네트워크(수천만의 경로 수)에 대응하도록 설계되었다. 그래서 BGP로 동작하는 라우터는 비교적 고가인 제품이 많다.

AS(Autonomous System)는 하나의 정책을 바탕으로 관리되는 네트워크(자율 시스템)를 말한다. ISP, 엔터프라이즈 기업, 공공기관 같은 조직이 이에 해당하며 인터넷은 이러한 자율 시스템의 집합체이다.

#### 구성 요소 확인하기
![image](https://user-images.githubusercontent.com/86287920/205193654-6847c158-447c-4dc6-b3c2-b641d8fe563c.png)

- Daemonset으로 각 노드에 calico-node Pod가 동작하여, 해당 파드에 bird, felix, confd등이 동작 + calico controller Pod는 Deployment로 생성
  - Calico의 특징은 BGP를 이용해 각 노드에 할당된 Pod 대역의 정보를 전달한다. 즉, 쿠버네티스 서버뿐만 아니라 물리적인 라우터와도 연동이 가능하다는 뜻이다. (Flannel의 경우 해당 구성 불가)
  - Calico Pod 안에서 Bird라고 하는 오픈소스 라우팅 데몬 프로그램이 프로세스로 동작하여 각 Node의 Pod 정보가 전파되는 것이다.
  - 이후 Felix라는 컴포넌트가 리눅스 라우터의 라우팅 테이블 및 iptables rule에 전달 받은 정보를 주입하는 형태이다.
  - confd는 변경되는 값을 계속 반영할 수 있도록 트리거하는 역할이다.

# Calico 기본 통신 과정

#### 동일 노드 내 파드 간 통신

간단히 말하면 동을 노드 내의 파드 간 통신은 내부에서 직접 통신된다.

![image](https://user-images.githubusercontent.com/86287920/205193831-239fdca1-928c-46dc-8279-7a549a418ae0.png)

Pod 생성하기 전 노드(k8s-w1)의 기본 상태

![image](https://user-images.githubusercontent.com/86287920/205193860-025fcf19-b5c4-4c5d-ae71-831903dc22b7.png)

하지만 이때 Pod를 2개 생성 해준다.
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  nodeSelector:
    alpha.eksctl.io/instance-id: i-03f1a15dd95c9b883
  containers:
  - name: pod1
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
    terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  nodeSelector:
    alpha.eksctl.io/instance-id: i-03f1a15dd95c9b883
  containers:
  - name: pod2
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
    terminationGracePeriodSeconds: 0  
```
그 뒤 calicoctl get workloadEndpoint 명령어를 사용해서 Pod의 endpoint를 확인해보자.

![image](https://user-images.githubusercontent.com/86287920/205194221-ab1f8257-99cc-469b-bb67-207de06ca4bc.png)

![image](https://user-images.githubusercontent.com/86287920/205194234-4825ed1c-2fed-486b-833c-25aa586d2fa5.png)

이를 k8s-w1 노드에 접근해서 Network Interface를 확인해보
