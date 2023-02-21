# Cluster Autoscaler

The Kubernetes [Cluster Autoscaler] automatically adjusts the number of nodes in your cluster when pods fail or are rescheduled onto other nodes. The Cluster Autoscaler is typically installed as a [Deployment] in your cluster. It uses [leader election] to ensure high availability, but scaling is done by only one replica at a time.

Before you deploy the Cluster Autoscaler, make sure that you're familiar with how Kubernetes concepts interface with AWS features. The following terms are used throughout this topic:

- **Kubernetes Cluster Autoscaler** – A core component of the Kubernetes control plane that makes scheduling and scaling decisions. For more information, see [Kubernetes Control Plane FAQ] on GitHub.

- **AWS Cloud provider implementation** – An extension of the Kubernetes Cluster Autoscaler that implements the decisions of the Kubernetes Cluster Autoscaler by communicating with AWS products and services such as Amazon EC2. For more information, see [Cluster Autoscaler on AWS] on GitHub.

- **Node groups** – A Kubernetes abstraction for a group of nodes within a cluster. Node groups aren't a true Kubernetes resource, but they're found as an abstraction in the Cluster Autoscaler, Cluster API, and other components. Nodes that are found within a single node group might share several common properties such as labels and taints. However, they can still consist of more than one Availability Zone or instance type.

- **Amazon EC2 Auto Scaling groups** – A feature of AWS that's used by the Cluster Autoscaler. Auto Scaling groups are suitable for a large number of use cases. Amazon EC2 Auto Scaling groups are configured to launch instances that automatically join their Kubernetes cluster. They also apply labels and taints to their corresponding node resource in the Kubernetes API.

For reference, [Managed node groups] are managed using Amazon EC2 Auto Scaling groups, and are compatible with the Cluster Autoscaler.

This topic describes how you can deploy the Cluster Autoscaler to your Amazon EKS cluster and configure it to modify your Amazon EC2 Auto Scaling groups.

----
[Cluster Autoscaler]: https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler
[Deployment]: https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler
[leader election]: https://en.wikipedia.org/wiki/Leader_election
[Kubernetes Control Plane FAQ]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md
[Cluster Autoscaler on AWS]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md
[Managed node groups]: https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html


## AutoScaling
```
curl -O https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
mv cluster-autoscaler-autodiscover.yaml CA.yaml
```

```vim CA.yaml```

![image](https://user-images.githubusercontent.com/86287920/201500846-5c8d41e8-d192-4c9c-9c2e-5b6874fa8b73.png)

![image](https://user-images.githubusercontent.com/86287920/201500849-b5e574de-de9e-4b4b-8389-8a401c3ceba6.png)

![image](https://user-images.githubusercontent.com/86287920/201500850-b98e1d01-8021-4701-80e2-f9604ca1061c.png)

``` kubectl apply -f CA.yaml ```
