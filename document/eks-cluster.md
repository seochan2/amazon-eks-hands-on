# 3. Create EKS Cluster

### 3.1 Create EKS Cluster with eksctl
#### 3.1.1 Create a eks-demo-cluster.yaml
```
cd ~/environment
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
    instanceType: m5.large # Instance type for node group
    desiredCapacity: 3 # The number of worker node in EKS Cluster
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

#### 3.1.3 Check that the node is properly deployed.
```
kubectl get nodes 
```
