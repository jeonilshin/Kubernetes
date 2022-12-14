# Create EKS Cluster using eksctl
### Installing eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### cluster.yaml
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: skills-cluster
  region: ap-northeast-2
  version: "1.22"
cloudWatch:
  clusterLogging:
    enableTypes: ["*"]
iam:
  withOIDC: true
vpc:
  id:
  subnets:
    public:
      ap-northeast-2a-pub: { id: $subnet_id }
      ap-northeast-2b-pub: { id: $subnet_id }
      ap-northeast-2c-pub: { id: $subnet_id }
    private:
      ap-northeast-2a-priv: { id: $subnet_id }
      ap-northeast-2b-priv: { id: $subnet_id }
      ap-northeast-2c-priv: { id: $subnet_id }
managedNodeGroups:
  - name: skills-app
    instanceName: skills-app
    labels: { skills/dedicated: app }
    tags: { skills/dedicated: app }
    instanceType: c5.large
    minSize: 2
    maxSize: 20
    desiredCapacity: 2
    privateNetworking: true
    subnets:
     - ap-northeast-2a-priv
     - ap-northeast-2b-priv
     - ap-northeast-2c-priv
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        awsLoadBalancerController: true
        cloudWatch: true
  - name: skills-addon
    instanceName: skills-addon
    labels: { skills/dedicated: addon }
    tags: { skills/dedicated: addon }
    instanceType: c5.large
    minSize: 2
    maxSize: 20
    desiredCapacity: 2
    privateNetworking: true
    subnets:
     - ap-northeast-2a-priv
     - ap-northeast-2b-priv
     - ap-northeast-2c-priv
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        awsLoadBalancerController: true
        cloudWatch: true
```

``` eksctl create cluster -f cluster.yaml ```

``` 
aws eks update-kubeconfig --region ap-northeast-2 --name <YOUR-CLUSTER-NAME>
```
**USE THIS WHEN AWS IS NOT CONFIGURED**
