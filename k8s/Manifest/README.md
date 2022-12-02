# This are the Manifest files that I prepared for Kubernetes Projects
# Namespace
```
apiVersion: v1
kind: Namespace

metadata:
  name: skills
```
# Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: match-deployment
spec:
  selector:
    matchLabels:
      skills/version: v1
      type: match
  replicas: 2
  template:
    metadata:
      labels:
        skills/version: v1
        type: match
    spec:
      containers:
      - name: match-container
        image: 956193760179.dkr.ecr.ap-northeast-2.amazonaws.com/match-ecr:latest
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```
# Services
```
apiVersion: v1
kind: Service

metadata:
  name: skills-cloud-service-stress
  namespace: skills
spec:
  selector:
    skills/version: v1
    type: stress
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  type: NodePort
# Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: match
  namespace: skills
  annotations:
    alb.ingress.kubernetes.io/load-balancer-name: match
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: skills-public-a, skills-public-b, skills-public-c
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/target-node-labels: skills/dedicated=app
    alb.ingress.kubernetes.io/actions.response-403: >
      {"type":"fixed-response","fixedResponseConfig":{"contentType":"text/plain","statusCode":"403","messageBody":"403 error text"}}
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: "/v1/"
        pathType: Prefix
        backend:
          service:
            name: match
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: response-403
            port:
              name: use-annotation

```