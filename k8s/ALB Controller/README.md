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
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```
```
helm repo add eks https://aws.github.io/eks-charts
```
```
helm repo update
```
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller 
```
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![image](https://user-images.githubusercontent.com/86287920/204777930-cde4c866-dd6c-4490-b9a9-608fc10629e7.png)
```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```
##### aws controller logs
```
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer-controller[a-zA-Z0-9-]+")
```
