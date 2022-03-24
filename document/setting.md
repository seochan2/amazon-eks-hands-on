# 1. Setting workspace 

### 1.1 AWS Cloud9
#### 1.1.1 IDE configuration with AWS Cloud9
- Cloud9 console > Create environment > platform : Amazon Linux 2
- Create in a public subnet

#### 1.1.2 Create IAM Role
- Create an IAM Role with Administrator access

#### 1.1.3 Grant IAM Role to an AWS Cloud9 instance
- EC2 instnace console > Select AWS Cloud9 instance, Actions > Security > Modify IAM Role
- Change IAM role

#### 1.1.4 Update IAM settings in IDE
- Disable AWS Cloud9 credentials. After that attach the IAM Role(because they are not compatible with EKS IAM authentication)
- Cloud9 IDE > AWS SETTINGS in the sidebar > Credentials > Disable the AWS managed temperature credits 
- Remove existing credential files 
```
rm -vf ${HOME}/.aws/credentials
```
- Check that Cloud9 IDE is using the correct IAM Role
```
aws sts get-caller-identity --query Arn | grep eks-admin
```

### 1.2 AWS CLI
#### 1.2.1 Update AWS CLI
```
sudo pip install --upgrade awscli
```
#### 1.2.2 check the version
```
aws --version
```

### 1.3 kubectl
#### 1.3.1 Install kubectl
- Check to install the corresponding kubectl to the Amazon EKS version you want to deploy
  https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
```
sudo curl -o /usr/local/bin/kubectl  \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
```
```
sudo chmod +x /usr/local/bin/kubectl
```

### 1.4 etc
#### 1.4.1 Install jq
```
sudo yum install -y jq
```
#### 1.4.2 Install bash-completion
```
sudo yum install -y bash-completion
```

### 1.5 Install eksctl
#### 1.5.1 Install eksctl
- Download the latest eksctl binary 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
- Move the binary to the location /usr/local/bin
```
sudo mv -v /tmp/eksctl /usr/local/bin
```
- check the installation
```
eksctl version
```

### 1.6 AWS Cloud9 Additional Settings
#### 1.6.1 Set default value to AWS Region
```
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')

echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
    
aws configure set default.region ${AWS_REGION}
```
#### 1.6.2 Register the account ID 
```
export ACCOUNT_ID=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.accountId')

echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
```
