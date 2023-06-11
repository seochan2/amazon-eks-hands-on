# 3. EKS 클러스터 생성하기
### 3.1 eksctl을 사용하여 EKS 클러스터 생성하기
#### 3.1.1 작업 폴더 이동
```
cd ~/environment

mkdir manifests

cd manifests
```

#### 3.1.2 클러스터 배포
```
cat << EOF > ${EKS_CLUSTER_NAME}.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ${EKS_CLUSTER_NAME} # 생성할 EKS 클러스터명
  region: ${AWS_REGION} # 클러스터를 생성할 리전
  version: "1.26" # 클러스터 버전

vpc:
  subnets:
    private:
      ${EKS_AZS_AZ1}: { id: ${EKS_AZS_ID1} } # Private Subnet1 ID
      ${EKS_AZS_AZ2}: { id: ${EKS_AZS_ID2} } # Private Subnet2 ID

managedNodeGroups:
  - name: node-group # 클러스터의 노드 그룹명
    instanceType: m5.large # 클러스터 워커 노드의 인스턴스 타입
    desiredCapacity: 2 # 클러스터 워커 노드의 갯수
    volumeSize: 20  # 클러스터 워커 노드의 EBS 용량 (단위: GiB)
    privateNetworking: true
    ssh:
      enableSsm: true
    iam:
      withAddonPolicies:
        imageBuilder: true # Amazon ECR에 대한 권한 추가
        albIngress: true  # albIngress에 대한 권한 추가
        cloudWatch: true # cloudWatch에 대한 권한 추가
        autoScaler: true # auto scaling에 대한 권한 추가
        ebs: true # EBS CSI Driver에 대한 권한 추가

cloudWatch:
  clusterLogging:
    enableTypes: ["*"]

iam:
  withOIDC: true
EOF
```

#### 3.1.3 노드 배포 확인
```
kubectl get nodes 
```

#### 3.1.4 결과 확인
```
NAME                                             STATUS   ROLES    AGE    VERSION
ip-10-0-133-67.ap-northeast-2.compute.internal   Ready    <none>   7m6s   v1.26.4-eks-0a21954
ip-10-0-151-81.ap-northeast-2.compute.internal   Ready    <none>   7m6s   v1.26.4-eks-0a21954
```
