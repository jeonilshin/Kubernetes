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
  name: stress-deployment
  namespace: skills
spec:
  selector:
    matchLabels:
      type: stress
  replicas: 2
  template:
    metadata:
      labels:
        skills/version: v1
        type: stress
    spec:
      containers:
      - name: stress-container
        image: 956193760179.dkr.ecr.ap-northeast-2.amazonaws.com/stress-ecr:latest
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
  name: stress-service
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
```
# Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: stress-ingress
  namespace: skills
  annotations:
    alb.ingress.kubernetes.io/load-balancer-name: stress
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: skills-private-a, skills-private-b, skills-private-c
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
            name: stress-service
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
# HPA (Horizontal Pod Autoscaler)
```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: stress-hpa
  namespace: skills
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stress-deployment
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```
