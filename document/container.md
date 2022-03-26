# 2. Container Image

### Upload container image to Amazon ECR
#### Create Amazon ECR Repository and Upload Image
- Download the source code to be containerized 
```
git clone https://github.com/joozero/amazon-eks-flask.git
``` 

#### Create an image repository
```
aws ecr create-repository --repository-name demo-flask-backend --image-scanning-configuration scanOnPush=true --region ${AWS_REGION}
```

#### Bring the authentication token and push the container image to the repository
```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

#### Build the docker image
```
cd ~/environment/amazon-eks-flask

docker build -t demo-flask-backend .
```

#### Enable a docker image tag
```
docker tag demo-flask-backend:latest $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/demo-flask-backend:latest
```

#### Push the image into the repository
```
docker push $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/demo-flask-backend:latest
```
