# 5. Deploy Microservices

### 5.1 Deploy First Backend Service
* To proceed, "2. Container Image" to Amazon ECR part must be preceded.

#### 5.1.1 Move on to manifests folder
```
cd ~/environment/manifests/
```

#### 5.1.2 Create deloy manifest
```
cat <<EOF> backend-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-nodejs-backend
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-nodejs-backend
  template:
    metadata:
      labels:
        app: sample-nodejs-backend
    spec:
      containers:
        - name: sample-nodejs-backend
          image: $ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/sample-nodejs-backend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
EOF
```

#### 5.1.3 Create service manifest
```
cat <<EOF> backend-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: sample-nodejs-backend
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/"
spec:
  selector:
    app: sample-nodejs-backend
  type: NodePort
  ports:
    - port: 8080 
      targetPort: 8080 
      protocol: TCP
EOF
```

#### 5.1.4 Create ingress manifest
```
cat <<EOF> backend-ingress.yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: "backend-ingress"
    namespace: default
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internet-facing
      alb.ingress.kubernetes.io/target-type: ip
spec:
    rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: "sample-nodejs-backend"
                port:
                  number: 8080
EOF
```

#### 5.1.5 Deploy the manifest created above in the order shown below. Ingress provisions Application Load Balancer(ALB)
```
kubectl apply -f backend-deployment.yaml
```

- Check the backend-deployment
```
kubectl get pod
```

```
kubectl apply -f backend-service.yaml
```
- Check the backend-service
```
kubectl get svc
```


```
kubectl apply -f backend-ingress.yaml
```
- Check the backend-ingress
```
kubectl get ingress
```

#### 5.1.6 Check that ALB endpoint
```
echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/contents/aws
```


### 5.2 Deploy Second Backend Service

#### 5.2.1 Get the backend endpoint
```
echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')
```

### 5.2.2 Modify the backend url
```
cd ~/environment/sample-react-app
```
```
vi nginx.conf
```
- Replace http://aaa.bbb.ccc with backend url

#### 5.2.3 Build
```
npm run build
```

#### 5.2.4 Push the image into the repository
```
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com
```
```
docker build -t sample-react-app .
```
```
docker tag sample-react-app:latest $ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/sample-react-app:latest
```
```
docker push $ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/sample-react-app:latest
```
#### 5.2.5 Delete the container image
```
docker images -a
```

- Result example
```
REPOSITORY                                                           TAG          IMAGE ID       CREATED         SIZE
<none>                                                               <none>       85fb9a09b553   2 minutes ago   143MB
<none>                                                               <none>       e8c96d7d909e   2 minutes ago   143MB
876630244803.dkr.ecr.ap[####].com/sample-react-app                 latest       6e84f53bb0ea   2 minutes ago   143MB
sample-react-app                                                     latest       6e84f53bb0ea   2 minutes ago   143MB
<none>                                                               <none>       991ab0515040   2 minutes ago   143MB
<none>                                                               <none>       e956badfd97e   2 minutes ago   142MB
<none>                                                               <none>       159cef29cf02   2 minutes ago   143MB
<none>                                                               <none>       0f79de9797c8   2 minutes ago   142MB
<none>                                                               <none>       f04b20055c8c   2 minutes ago   142MB
<none>                                                               <none>       d0fb434ea76d   2 minutes ago   142MB
node                                                                 12-alpine    8dc0ee810c0a   2 days ago      91MB
```

- Copy the Image ID of the 876630244803.dkr.ecr.ap[####].com/sample-react-app 

```
docker rmi --force 6e84f53bb0ea
```

#### 5.2.6 Move to manifests
```
cd ~/environment/manifests/
```

#### 5.2.7 Create the deploy manifest
- replace ap-northeast-1 with your region
```
cat <<EOF> frontend-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-react-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-react-app
  template:
    metadata:
      labels:
        app: sample-react-app
    spec:
      containers:
        - name: sample-react-app
          image: $ACCOUNT_ID.dkr.ecr.ap-northeast-1.amazonaws.com/sample-react-app:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
EOF
```

#### 5.2.8 Create the service manifest 

```
cat <<EOF> frontend-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: sample-react-app
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/"
spec:
  selector:
    app: sample-react-app
  type: NodePort
  ports:
    - port: 80 # 서비스가 생성할 포트  
      targetPort: 80 # 서비스가 접근할 pod의 포트
      protocol: TCP
EOF
```

#### 5.2.9 Create the ingress manifest
```
cat <<EOF> frontend-ingress.yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: "frontend-ingress"
    namespace: default
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/scheme: internet-facing
      alb.ingress.kubernetes.io/target-type: ip
spec:
    rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: "sample-react-app"
                port:
                  number: 80
EOF
```

#### 5.2.10 provisioning the ALB
```
kubectl apply -f frontend-deployment.yaml
```

- Check the frontend-deployment
```
kubectl get pod
```

```
kubectl apply -f frontend-service.yaml
```
- Check the frontend-service
```
kubectl get svc
```

```
kubectl apply -f frontend-ingress.yaml
```
- Check the frontend-ingress
```
kubectl get ingress
```

#### 5.2.11 Check that ALB endpoint
```
echo http://$(kubectl get ingress/frontend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/
```
