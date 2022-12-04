# Amazon CloudWatch Container Insight

Use CloudWatch Container Insights to collect, aggregate, and summarize metrics and logs from your containerized applications and microservices. Container Insights is available for Amazon Elastic Container Service (Amazon ECS), Amazon Elastic Kubernetes Service (Amazon EKS), and Kubernetes platforms on Amazon EC2. Amazon ECS support includes support for Fargate.

CloudWatch automatically collects metrics for many resources, such as CPU, memory, disk, and network. Container Insights also provides diagnostic information, such as container restart failures, to help you isolate issues and resolve them quickly. You can also set CloudWatch alarms on metrics that Container Insights collects.

## CloudWatch metric
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cloudwatch-namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-serviceaccount.yaml
curl -O https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-configmap.yaml
vim cwagent-configmap.yaml
```
```
    {
      "agent": {
        "region": "<REGION>"
      },
      "logs": {
        "metrics_collected": {
          "kubernetes": {
            "cluster_name": "<CLUSTER-NAME>",
            "metrics_collection_interval": 60
          }
        },
        "force_flush_interval": 5,
        "endpoint_override": "logs.ap-northeast-2.amazonaws.com"
      },
      "metrics": {
        "metrics_collected": {
          "statsd": {
            "service_address": ":8125"
          }
        }
      }
    }
```

![image](https://user-images.githubusercontent.com/86287920/205487942-e67c77e3-6938-4fcd-9a6c-a7a89f50ea35.png)

```
kubectl apply -f cwagent-configmap.yaml
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-daemonset.yaml
```
