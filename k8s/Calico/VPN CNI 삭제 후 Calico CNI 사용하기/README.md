# VPN CNI 삭제 후 Calico CNI 사용하기

AWS EKS에 Calico를 사용하는 방법으로는 두 가지 가 있다.
  1. Calico를 Network Policy로만 사용하기
  2. Calico를 Network Policy + CNI + Routing 용도로 사용하기

2번 Calico를 Network Policy + CNI + Routing 용도로 사용해보자

**※ 해당 작업은 Node가 없는 상태에서 진행해야 됨. 즉, 과제 수행 시 EKS Cluster만 생성하고 CNI 교체 한 뒤 Node Group을 생성해줘야 한다.**

우선 기존에 설치 되어있는 VPC CNI를 삭제해준다.
``` kubectl delete daemonset -n kube-system aws-node ```

그리고 Calico CNI를 설치해준다.
``` kubectl apply -f https://docs.projectcalico.org/manifests/calico-vxlan.yaml ```

Now Lets 생성 node Group
``` eksctl create nodegroup --cluster my-calico-cluster --node-type t3.medium --max-pods-per-node 100 ```
