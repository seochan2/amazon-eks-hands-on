# 4. Create Ingress Controller

### Create AWS Load Balancer Controller
#### Create a folder named manifests 
- /home/ec2-user/environment/manifests/alb-ingress-controller
```
cd ~/environment

mkdir -p manifests/alb-ingress-controller && cd manifests/alb-ingress-controller
```

#### Create IAM OpenID Connect (OIDC) identity provider for the cluster
```
eksctl utils associate-iam-oidc-provider --region ${AWS_REGION} --cluster eks-demo --approve
```

#### Create an IAM Policy to grant to the AWS Load Balancer Controller
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

#### Create ServiceAccount for AWS Load Balancer Controller
```
eksctl create iamserviceaccount \
    --cluster eks-demo \
    --namespace kube-system \
    --name aws-load-balancer-controller \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

#### Add AWS Load Balancer controller to the cluster
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.4.1/cert-manager.yaml
```

#### Download Load balancer controller yaml file
```
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/v2_2_1_full.yaml
```

#### In yaml file, edit cluster-name to eks-demo
```
spec:
    containers:
    - args:
        - --cluster-name=eks-demo # Insert EKS cluster that you created
        - --ingress-class=alb
        image: amazon/aws-alb-ingress-controller:v2.2.1
```

#### Remove the ServiceAccount yaml spec written in the yaml file
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/name: aws-load-balancer-controller
  name: aws-load-balancer-controller
  namespace: kube-system
```

#### Deploy AWS Load Balancer controller file
```
kubectl apply -f v2_2_1_full.yaml
```

#### Check that the deployment is successed 
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
