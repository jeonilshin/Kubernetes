# ☸ KUBERNETES RBAC
<p align="center"><img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100"></p>

``` role.yaml ```
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: 역할 이름
  namespace: 네임스페이스
rules:
  - apiGroups:
      - ""
      - "apps"
      - "batch"
      - "extensions"
    resources:
      - "configmaps"
      - "cronjobs"
      - "deployments"
      - "events"
      - "ingresses"
      - "jobs"
      - "pods"
      - "pods/attach"
      - "pods/exec"
      - "pods/log"
      - "pods/portforward"
      - "secrets"
      - "services"
    verbs:
      - "create"
      - "delete"
      - "describe"
      - "get"
      - "list"
      - "patch"
      - "update"
```
``` rolebinding.yaml ```
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: 이름-rolebinding
  namespace: 네임스페이스
subjects:
- kind: User
  name: 역할-user
roleRef:
  kind: Role
  name: 역할
  apiGroup: rbac.authorization.k8s.io
```
### Map the IAM Role of aws-auth ConfigMap to RBAC Role and Group 
```
eksctl create iamidentitymapping --cluster worldskills-cloud-cluster --arn [역할의 ARN] --username worldskills-cloud-control-role-user
```
## Test out the Role Security
```
kubectl get pods -n 네임스페이스
kubectl get pods -n default
```
![image](https://user-images.githubusercontent.com/86287920/199452998-a36b7135-7976-4728-95ce-0a60ace3e57b.png)

```
kubectl create job hello -n worldskills-ns --image=busybox -- echo "Hello World"
```
![image](https://user-images.githubusercontent.com/86287920/199453095-2bbc8660-d7a3-484c-8301-bf455da0a617.png)

```
kubectl create job hello -n default --image=busybox -- echo "Hello World"
```
![image](https://user-images.githubusercontent.com/86287920/199454059-df0765a7-7806-4fb7-b741-9f056ab54067.png)
