# 1. 실습 환경 구축

### 1.1 AWS Cloud9 구성
#### 1.1.1 VPC 생성
- AWS Console > VPC > Create VPC > VPC Setting
- Resources to create : VPC and more
- Name tag auto-generation : eks
- IPv4 CIDR block : 기본값(10.0.0.0/16)
- IPv6 CIDR block : No IPv6 CIDR block
- Tenancy : 기본값(Default)
- Number of Availability Zones (AZs) : 기본값(2)
- Number of public subnets : 기본값(2)
- Number of private subnets : 기본값(2)
- NAT gateways ($) : In 1 AZ
- VPC endpoints : 기본값(S3 Gateway)
- DNS options : 기본값(Enable DNS hostnames / Enable DNS resolution)
- Additional tags - Add new tag : Key (UserType) , Value(eks) 
- Create VPC

#### 1.1.2 AWS Cloud9으로 IDE 구성
Cloud9 기반의 실습 환경 구축
- AWS Console > Cloud9 > Create environment
- Name 입력 : eks-workspace-<닉네임> (ex, eks-workspace-scp)
- Environment type : 기본값(New EC2 instance)
- Instance Type : t3.small
- Platform : 기본값(Amazon Linux 2)
- Timeout : 기본값(30 minutes)
- Connection : Secure Shell(SSH)
- VPC Setting : Amazon Virtual Private Cloud (VPC) : 1.1.1에서 생성한 VPC / Subnet : public Subnet 선택(eks-subnet-public~)
- Create 

#### 1.1.3 IAM Role 생성
Administrator access 정책을 가진 IAM Role을 생성
- AWS Console > IAM > ROLE > Create Role
- Trusted entity type : AWS service
- Use case : EC2 > Next
- AdministratorAccess 검색후 선택 > Next
- Role Name : eks-workspace-admin-<닉네임> (ex. eksworkspace-admin-scp) / Role Name에 "." 주의
- Create Role

#### 1.1.4 AWS Cloud9 Instance에 IAM Role 부여
실습 환경에 Admin 권한 부여 
- EC2 instnace console > Select AWS Cloud9 instance (aws-cloud9-eks-workspace-44178fc24ede445193e90242705c4c11 형식)
- Actions > Security > Modify IAM Role 선택
- Change IAM role 리스트에서 1.1.3에서 생성한 Role Name(ex. eksworkspace-admin-scp) 선택
- Update IAM role

#### 1.1.5 IDE에서 IAM 설정 업데이트
Cloud9의 기본 IAM credentials은 동적으로 관리되고(15분 간격으로 refresh),  EKS IAM authentication과 호환되지 않기에 이를 비활성화하고 IAM Role을 붙임
- Cloud9 IDE > 우측 상단 기어 아이콘 클릭 > Preferences > AWS Setting - Disable the AWS managed temperature credits 

- 기존의 자격 증명 파일도 제거(선택)
```
rm -vf ${HOME}/.aws/credentials
```
- Cloud9 IDE가 올바른 IAM Role을 사용하고 있는지 확인
```
aws sts get-caller-identity --query Arn | grep eks-workspace-admin-<닉네임>
```
- 결과 예시   
`arn:aws:sts::876630244803:assumed-role/eksworkspace-admin-scp/i-03e9af70279e891de`


### 1.2 AWS CLI
#### 1.2.1 AWS CLI 1.x 버전 삭제
```
sudo rm /usr/bin/aws

sudo rm /usr/bin/aws_completer

sudo rm -rf /usr/local/aws-cli
```

#### 1.2.2 AWS CLI 2.x 버전 설치
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

export PATH=/usr/local/bin:$PATH

source ~/.bash_profile
```

#### 1.2.3 버전 확인 (2.x대 버전)
```
aws --version
```
- 결과 예시   
`aws-cli/2.11.25 Python/3.11.3 Linux/4.14.314-237.533.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off`

### 1.3 kubectl
#### 1.3.1 kubectl 최신 바이너리 다운로드
[설치 페이지](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html)를 참조하여, 배포할 Amazon EKS 버전과 상응하는 kubectl를 설치(최신버전 : 2023-06-01 기준, 1.26.4)
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl
```

#### 1.3.2 바이너리 실행 권한 적용 및 패스 설정
```
chmod +x ./kubectl

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
```

#### 1.3.3 버전 확인
```
kubectl version --short --client
```
- 결과 예시   
`Client Version: v1.26.4-eks-0a21954`   
`Kustomize Version: v4.5.7`

### 1.4 eksctl 
#### 1.4.1 eksctl 최신 바이너리 다운로드 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```

#### 1.4.2 바이너리 폴더 변경
```
sudo mv -v /tmp/eksctl /usr/local/bin
```

#### 1.4.3 버전 확인
```
eksctl version
```
- 결과 예시   
`0.144.0`

### 1.5 기타 툴
#### 1.5.1 jq 설치
JSON 형식의 데이터를 다루는 커맨드라인 유틸리티
```
sudo yum install -y jq
```

#### 1.5.2 bash-completion 설치
kubectl 명령어의 자동 완성 지원
```
sudo yum install -y bash-completion
```

#### 1.5.3 K9s 설치
터미널에서 Kubernetes 클러스터를 관리할 수 있는 CLI 도구구
```
K9S_VERSION=$(curl -s https://api.github.com/repos/derailed/k9s/releases/latest | jq -r '.tag_name')

curl -sL https://github.com/derailed/k9s/releases/download/${K9S_VERSION}/k9s_Linux_amd64.tar.gz | sudo tar xfz - -C /usr/local/bin k9s
```
#### 1.5.3.1 버전 확인
```
k9s version
```
- 결과 예시   
`  ____  __.________      `   
`|    |/ _/   __   \______`   
`|      < \____    /  ___/`   
`|    |  \   /    /\___ \ `   
`|____|__ \ /____//____  >`   
`        \/            \/ `   
`Version:    v0.27.4`   
`Commit:     f4543e9bd2f9e2db922d12ba23363f6f7db38f9c`   
`Date:       2023-05-07T16:55:34Z`

#### 1.5.4 Helm 설치
쿠버네티스를 위한 패키지 매니저
```
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

#### 1.5.4.1 버전 확인
```
helm version
```

- 결과 예시   
`version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.3"}`

### 1.6 Cloud9 추가 설정
#### 1.6.1 기본 리전 설정
```
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
   
aws configure set default.region ${AWS_REGION}
```

#### 1.6.2 계정 ID 환경변수 등록
```
export ACCOUNT_ID=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.accountId')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
```

#### 1.6.3 EKS 클러스터명 환경변수 등록
```
export EKS_CLUSTER_NAME=eks-scp-cluster

echo "export EKS_CLUSTER_NAME=${EKS_CLUSTER_NAME}" | tee -a ~/.bash_profile

export EKS_AZS_AZ1=($(aws ec2 describe-subnets --filters "Name=tag:UserType,Values=eks" "Name=tag:Name,Values=*private*" --query 'Subnets[].[AvailabilityZone][0]' --output text --region $AWS_REGION))

export EKS_AZS_AZ2=($(aws ec2 describe-subnets --filters "Name=tag:UserType,Values=eks" "Name=tag:Name,Values=*private*" --query 'Subnets[].[AvailabilityZone][1]' --output text --region $AWS_REGION))

export EKS_AZS_ID1=($(aws ec2 describe-subnets --filters "Name=tag:UserType,Values=eks" "Name=tag:Name,Values=*private*" --query 'Subnets[].[SubnetId][0]' --output text --region $AWS_REGION))

export EKS_AZS_ID2=($(aws ec2 describe-subnets --filters "Name=tag:UserType,Values=eks" "Name=tag:Name,Values=*private*" --query 'Subnets[].[SubnetId][1]' --output text --region $AWS_REGION))

echo "export EKS_AZS_AZ1=${EKS_AZS_AZ1}" | tee -a ~/.bash_profile

echo "export EKS_AZS_AZ2=${EKS_AZS_AZ2}" | tee -a ~/.bash_profile

echo "export EKS_AZS_ID1=${EKS_AZS_ID1}" | tee -a ~/.bash_profile

echo "export EKS_AZS_ID2=${EKS_AZS_ID2}" | tee -a ~/.bash_profile
```
#### 1.6.4 Cloud9 디스크 사이즈 증설
```
wget https://gist.githubusercontent.com/joozero/b48ee68e2174a4f1ead93aaf2b582090/raw/2dda79390a10328df66e5f6162846017c682bef5/resize.sh

sh resize.sh
```

#### 1.6.4.1 결과 확인
```
df -h
```
- 결과 예시   
`Filesystem      Size  Used Avail Use% Mounted on`   
`devtmpfs        3.8G     0  3.8G   0% /dev`   
`tmpfs           3.8G     0  3.8G   0% /dev/shm`   
`tmpfs           3.8G  460K  3.8G   1% /run`   
`tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup`   
`/dev/nvme0n1p1   30G  7.1G   23G  24% /`   
`tmpfs           776M     0  776M   0% /run/user/1000`   
