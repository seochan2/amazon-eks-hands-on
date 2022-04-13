# 6. Autoscaling Pod & Cluster

### 6.0 Kubernetes Auto Scaling
```
Kubernetis has two main auto-scaling capabilities
- HPA(Horizontal Pod AutoScaler)
- Cluster Autoscaler

HPA automatically scales the number of pods by observing CPU usage or custom metrics. 
However, if you run out of EKS cluster's own resources to which the pod goes up, consider Cluster Autoscaler.
```

### 6.1 Apply HPA
#### 6.1.1 Create metrics server
* Metrics Server aggregates resource usage data across the Kubernetes cluster
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```

#### 6.1.2 Check that the metrics server is created successfully
```
kubectl get deployment metrics-server -n kube-system
```

#### 6.1.3 Modify flask deployment yaml file
* Yaml file that you created in Deploy First Backend Service
* Change replicas to 1 and set the amount of resources required for the container
```
cd /home/ec2-user/environment/manifests
```
```
cat <<EOF> flask-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-flask-backend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-flask-backend
  template:
    metadata:
      labels:
        app: demo-flask-backend
    spec:
      containers:
        - name: demo-flask-backend
          image: $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/demo-flask-backend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 250m
            limits:
              cpu: 500m
EOF
```

#### 6.1.4 Apply the yaml file
```
kubectl apply -f flask-deployment.yaml
```

#### 6.1.5 Create the HPA yaml file
```
cat <<EOF> flask-hpa.yaml
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: demo-flask-backend-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demo-flask-backend
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 30
EOF
```

#### 6.1.6 Deploy yaml file
```
kubectl apply -f flask-hpa.yaml
```

#### 6.1.7 Check the HPA status 
```
kubectl get hpa
```

* Result
```
NAME                     REFERENCE                       TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
demo-flask-backend-hpa   Deployment/demo-flask-backend   4%/30%    1         5         1          36s
``
