# 3. Create EKS Cluster

### 3.1 Create EKS Cluster with eksctl
#### 3.1.1 Create a eks-demo-cluster.yaml
```
cd ~/environment
```
```
mkdir manifests
```
```
cd manifests
```
```
cat << EOF > eks-demo-cluster.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-demo # EKS Cluster name
  region: ${AWS_REGION} # Region Code to place EKS Cluster
  version: "1.21"

vpc:
  cidr: "192.168.0.0/16" # CIDR of VPC for use in EKS Cluster

managedNodeGroups:
  - name: node-group # Name of node group in EKS Cluster
    instanceType: t3.small # Instance type for node group
    desiredCapacity: 2 # The number of worker node in EKS Cluster
    volumeSize: 10  # EBS Volume for worker node (unit: GiB)
    ssh:
      enableSsm: true
    iam:
      withAddonPolicies:
        imageBuilder: true # Add permission for Amazon ECR
        # albIngress: true  # Add permission for ALB Ingress
        cloudWatch: true # Add permission for CloudWatch
        autoScaler: true # Add permission Auto Scaling

cloudWatch:
  clusterLogging:
    enableTypes: ["*"]
EOF
```

#### 3.1.2 Deploy the cluster
```
eksctl create cluster -f eks-demo-cluster.yaml
```
- It takes about 20 minutes

#### 3.1.3 Check that the node is properly deployed.
```
kubectl get nodes 
```
- Example output
```
NAME                                                STATUS   ROLES    AGE   VERSION
ip-192-168-18-99.ap-northeast-1.compute.internal    Ready    <none>   90s   v1.21.5-eks-9017834
ip-192-168-70-110.ap-northeast-1.compute.internal   Ready    <none>   85s   v1.21.5-eks-9017834
```

#### 3.1.4 Check that Credential
```
cat ~/.kube/config
```
