<h1 align="center"> What is Calico? </h1>
<p align="center"><img src="https://user-images.githubusercontent.com/86287920/205188723-638b479b-da28-4ac4-aed8-ab30683db49d.png" width="100"></p>

[Calico] is an open source networking and network security solution for containers, virtual machines, and native host-based workloads. Calico supports a broad range of platforms including Kubernetes, OpenShift, Mirantis Kubernetes Engine (MKE), OpenStack, and bare metal services.

Whether you opt to use Calico's eBPF data plane or Linux’s standard networking pipeline, Calico delivers blazing fast performance with true cloud-native scalability. Calico provides developers and cluster operators with a consistent experience and set of capabilities whether running in public cloud or on-prem, on a single node, or across a multi-thousand node cluster.

----
[Calico]: https://projectcalico.docs.tigera.io/about/about-calico

# Installing Calico
```
wget https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-operator.yaml
wget https://raw.githubusercontent.com/aws/amazon-vpc-cni-k8s/master/config/master/calico-crs.yaml
```
```
kubectl apply -f calico-operator.yaml
kubectl apply -f calico-crs.yaml
```
### View the resources in the ```calico-system``` namespace.
```
kubectl get daemonset calico-node --namespace calico-system
```
![image](https://user-images.githubusercontent.com/86287920/205189085-16aa78c4-3009-4bd6-8b33-a4a92ebb9dbd.png)

# Network Policy (calico)
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: stress-network
  namespace: skills
spec:
  podSelector:
    matchLabels:
      type: stress
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - port: 443
        - port: 80
        - port: 53
          protocol: TCP
        - port: 53
          protocol: UDP
```
