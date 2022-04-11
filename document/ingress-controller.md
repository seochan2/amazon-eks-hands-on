# 4. Create Ingress Controller

### 4.1 Create AWS Load Balancer Controller
#### 4.1.1 Create a folder named manifests 
- /home/ec2-user/environment/manifests/alb-ingress-controller
```
cd ~/environment

mkdir -p manifests/alb-ingress-controller && cd manifests/alb-ingress-controller
```

#### 4.1.2 Create IAM OpenID Connect (OIDC) identity provider for the cluster
```
eksctl utils associate-iam-oidc-provider --region ${AWS_REGION} --cluster eks-demo --approve
```

- Get the cluster's OIDC provider url
```
aws eks describe-cluster --name eks-demo --query "cluster.identity.oidc.issuer" --output text
```

- heck the ID value from the url
```
aws iam list-open-id-connect-providers | grep 7C9832F25C000000000000C3
```

- Example output
```
"Arn": "arn:aws:iam::876630244803:oidc-provider/oidc.eks.ap-northeast-1.amazonaws.com/id/7C9832F25C000000000000C3"
```

#### 4.1.3 Create an IAM Policy to grant to the AWS Load Balancer Controller
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

#### 4.1.4 Create ServiceAccount for AWS Load Balancer Controller
```
eksctl create iamserviceaccount \
    --cluster eks-demo \
    --namespace kube-system \
    --name aws-load-balancer-controller \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

#### 4.1.5 Add AWS Load Balancer controller to the cluster
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.4.1/cert-manager.yaml
```

#### 4.1.6 Download Load balancer controller yaml file
```
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.1/docs/install/v2_2_1_full.yaml
```

#### 4.1.7 In yaml file, edit cluster-name to eks-demo
```
vi v2_2_1_full.yaml
```
```
spec:
    containers:
    - args:
        - --cluster-name=eks-demo # Insert EKS cluster that you created
        - --ingress-class=alb
        image: amazon/aws-alb-ingress-controller:v2.2.1
```

#### 4.1.8 Remove the ServiceAccount yaml spec written in the yaml file
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

#### 4.1.9 Deploy AWS Load Balancer controller file
```
kubectl apply -f v2_2_1_full.yaml
```

#### 4.1.10 Check that the deployment is successed 
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

- Example output
```
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   1/1     1            1           49s
```

#### 4.1.11 Check that service account has been created
```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```

#### 4.1.12 Check the relevant logs 
```
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")
```

#### 4.1.13 Check that detailed property values are available
```
ALBPOD=$(kubectl get pod -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")

kubectl describe pod -n kube-system ${ALBPOD}
```
