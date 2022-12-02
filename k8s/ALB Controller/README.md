# ALB Ingress Controller
1. First, create **IAM OpenID Connect(OIDC) identity provider** for the cluster. **IAM OIDC provider** must exist in the cluster in order for objects created by Kubernetes to use service account which purpose is to authenticate to API Server ro external services.
```
eksctl utils associate-iam-oidc-provider \
    --region ap-northeast-2 \
    --cluster worldskills-cloud-cluster \
    --approve
```
**if you have iam oidc allowed in cluster no need to do this**

2. Apply Helm Chart and Create an IAM Policy to grant to the AWS Load Balancer Controller
```
kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master
```
```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
3. Create ServiceAccount for AWS Load Balancer Controller
```
eksctl create iamserviceaccount \
    --cluster=<your-cluster> \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```
4. Add AWS Load Balancer controller to the cluster.
```
wget https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
```
``` kubectl apply -f cert-manager.yaml --validate=false ```
```
curl -Lo ALB.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.4/v2_4_4_full.yaml
```
```
sed -i.bak -e '480,488d' ./ALB.yaml
sed -i.bak -e 's|your-cluster-name|<YOUR-CLUSTER>|' ./ALB.yaml
```
```
kubectl apply -f ALB.yaml
kubectl apply -f https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.4/v2_4_4_ingclass.yaml
```
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```
##### aws controller logs
```
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")
```
![image](https://user-images.githubusercontent.com/86287920/204777930-cde4c866-dd6c-4490-b9a9-608fc10629e7.png)
