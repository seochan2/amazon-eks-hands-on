# 5. Deploy Microservices

### 5.1 Deploy First Backend Service
* To proceed, "2. Container Image" to Amazon ECR part must be preceded.

#### 5.1.1 Move on to manifests folder
```
cd ~/environment/manifests/
```

#### 5.1.2 Create deloy manifest
```
cat <<EOF> flask-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-flask-backend
  namespace: default
spec:
  replicas: 2
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
EOF
```

#### 5.1.3 Create service manifest

```
cat <<EOF> flask-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: demo-flask-backend
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/contents/aws"
spec:
  selector:
    app: demo-flask-backend
  type: NodePort
  ports:
    - port: 8080 
      targetPort: 8080 
      protocol: TCP
EOF
```

#### 5.1.4 Create ingress manifest

```
cat <<EOF> ingress.yaml
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
          - path: /contents
            pathType: Prefix
            backend:
              service:
                name: "demo-flask-backend"
                port:
                  number: 8080
EOF
```

#### 5.1.5 Deploy the manifest created above in the order shown below. Ingress provisions Application Load Balancer(ALB)
```
kubectl apply -f flask-deployment.yaml
```
```
kubectl apply -f flask-service.yaml
```
```
kubectl apply -f ingress.yaml
```

#### 5.1.6 Check that ALB endpoint
```
echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/contents/aws
```


### 5.2 Deploy Second Backend Service

#### 5.2.1 Move on to manifests folde
```
cd ~/environment/manifests/
```

### 5.2.2 Create deploy manifest
```
cat <<EOF> nodejs-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-nodejs-backend
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-nodejs-backend
  template:
    metadata:
      labels:
        app: demo-nodejs-backend
    spec:
      containers:
        - name: demo-nodejs-backend
          image: public.ecr.aws/y7c9e1d2/joozero-repo:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
EOF
```

#### 5.2.3 Create service manifest
```
cat <<EOF> nodejs-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: demo-nodejs-backend
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/services/all"
spec:
  selector:
    app: demo-nodejs-backend
  type: NodePort
  ports:
    - port: 8080
      targetPort: 3000
      protocol: TCP
EOF
```

#### 5.2.4 Modify ingress manifest file
* Modify ingress manifest file in 5.1.4 Deploy First Backend Service. 


```
cat <<EOF> ingress.yaml
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
          - path: /contents
            pathType: Prefix
            backend:
              service:
                name: "demo-flask-backend"
                port:
                  number: 8080
          - path: /services
            pathType: Prefix
            backend:
              service:
                name: "demo-nodejs-backend"
                port:
                  number: 8080
EOF
```

#### 5.2.5 ploy the manifest files 	
```
kubectl apply -f nodejs-deployment.yaml
```
```
kubectl apply -f nodejs-service.yaml
```
```
kubectl apply -f ingress.yaml
```

#### 5.2.6 Check that ALB endpoint	
```
echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/services/all
```


### 5.3 Deploy Frontend Service
#### 5.3.1 Download the source code
```
cd /home/ec2-user/environment
```
```
git clone https://github.com/joozero/amazon-eks-frontend.git
```

#### 5.3.2 Create an image repository
```
aws ecr create-repository \
--repository-name demo-frontend \
--image-scanning-configuration scanOnPush=true \
--region ${AWS_REGION}
```

#### 5.3.3 Change source code

* Change the url values in App.js file(/home/ec2-user/environment/amazon-eks-frontend/src/App.js)

```
$ echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/contents/'${search}'
```

* paste the values derived from the result 


#### 5.3.4 Change source code	

* Change the url values in upperPage.js file(/home/ec2-user/environment/amazon-eks-frontend/src/page/UpperPage.js)
```
$ echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')/services/all"
```

#### 5.3.5 install and build npm

```
cd /home/ec2-user/environment/amazon-eks-frontend
```
```
npm install
```
```
npm run build
```

#### 5.3.6 create container image repository and push image
```
docker build -t demo-frontend .
```
```
docker tag demo-frontend:latest $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/demo-frontend:latest
```
```
docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/demo-frontend:latest
```

#### 5.3.7 During applying above CLI, if you receive denied	message

* "denied: Your authorization token has expired. Reauthenticate and try again" then applying bottom command line and do this again.

```
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
```

#### 5.3.8 Move to manifests folder
```
cd /home/ec2-user/environment/manifests
```

#### 5.3.9 Create deployment yaml
```
cat <<EOF> frontend-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-frontend
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-frontend
  template:
    metadata:
      labels:
        app: demo-frontend
    spec:
      containers:
        - name: demo-frontend
          image: $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/demo-frontend:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80
EOF
```

#### 5.3.10 Create service yaml

```
cat <<EOF> frontend-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: demo-frontend
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/"
spec:
  selector:
    app: demo-frontend
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
```

#### 5.3.11 Create ingress yaml	
```
cat <<EOF> ingress.yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ""backend-ingress""
  namespace: default
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /contents
            pathType: Prefix
            backend:
              service:
                name: "demo-flask-backend"
                port:
                  number: 8080
          - path: /services
            pathType: Prefix
            backend:  
              service:
                name: "demo-nodejs-backend"
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: "demo-frontend"
                port:
                  number: 80
EOF
```

#### 5.3.12 Deploy manifest file
```
kubectl apply -f frontend-deployment.yaml
```
```
kubectl apply -f frontend-service.yaml
```
```
kubectl apply -f ingress.yaml
```

#### 5.3.13 Check that ALB endpoint	
```
echo http://$(kubectl get ingress/backend-ingress -o jsonpath='{.status.loadBalancer.ingress[*].hostname}')
```


* During applying above CLI, you receive error message. It's a simple data integrity issue, let's go once without solving it here

```
react-dom.production.min.js:209 
        
       TypeError: Cannot read properties of undefined (reading 'map')
    at k (UpperPage.js:50:25)
    at Qi (react-dom.production.min.js:153:146)
    at za (react-dom.production.min.js:175:309)
    at vl (react-dom.production.min.js:263:406)
    at su (react-dom.production.min.js:246:265)
    at lu (react-dom.production.min.js:246:194)
    at Zl (react-dom.production.min.js:239:172)
    at react-dom.production.min.js:123:115
    at t.unstable_runWithPriority (scheduler.production.min.js:19:467)
    at $o (react-dom.production.min.js:122:325)
el @ react-dom.production.min.js:209

Uncaught (in promise) TypeError: Cannot read properties of undefined (reading 'map')
    at k (UpperPage.js:50:25)
    at Qi (react-dom.production.min.js:153:146)
    at za (react-dom.production.min.js:175:309)
    at vl (react-dom.production.min.js:263:406)
    at su (react-dom.production.min.js:246:265)
    at lu (react-dom.production.min.js:246:194)
    at Zl (react-dom.production.min.js:239:172)
    at react-dom.production.min.js:123:115
    at t.unstable_runWithPriority (scheduler.production.min.js:19:467)
    at $o (react-dom.production.min.js:122:325)
```  
