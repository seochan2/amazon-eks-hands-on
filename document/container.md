# 2. Container Image

### 2.1 Upload container image to Amazon ECR (backend)
#### 2.1.1 Create Amazon ECR Repository and Upload Image
- Download the source code to be containerized 
```
cd ~/environment/
```
```
git clone https://github.com/sghaha/sample-nodejs-backend.git
``` 
```
npm install
```

#### 2.1.2 Create an image repository
```
aws ecr create-repository --repository-name sample-nodejs-backend --image-scanning-configuration scanOnPush=true --region ${AWS_REGION}
```

- Result example
```
{
    "repository": {
        "repositoryUri": "876630244803.dkr.ecr.ap-northeast-1.amazonaws.com/sample-nodejs-backend", 
        "imageScanningConfiguration": {
            "scanOnPush": true
        }, 
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }, 
        "registryId": "876630244803", 
        "imageTagMutability": "MUTABLE", 
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:876630244803:repository/sample-nodejs-backend", 
        "repositoryName": "sample-nodejs-backend", 
        "createdAt": 1649380735.0
    }
}
```

#### 2.1.3 Bring the authentication token and push the container image to the repository
```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

- Result example
```
WARNING! Your password will be stored unencrypted in /home/ec2-user/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

#### 2.1.4 Build the docker image
```
cd ~/environment/sample-nodejs-backend
```
```
docker build -t sample-nodejs-backend .
```

#### 2.1.5 Enable a docker image tag
```
docker tag sample-nodejs-backend:latest $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/sample-nodejs-backend:latest
```

#### 2.1.6 Push the image into the repository
```
docker push $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/sample-nodejs-backend:latest
```

#### 2.1.7 Delete the container image
```
docker images -a
```

- Result example
```
REPOSITORY                                                           TAG          IMAGE ID       CREATED         SIZE
<none>                                                               <none>       85fb9a09b553   2 minutes ago   143MB
<none>                                                               <none>       e8c96d7d909e   2 minutes ago   143MB
876630244803.dkr.ecr.ap[####].com/sample-nodejs-backend            latest       6e84f53bb0ea   2 minutes ago   143MB
sample-react-app                                                     latest       6e84f53bb0ea   2 minutes ago   143MB
<none>                                                               <none>       991ab0515040   2 minutes ago   143MB
<none>                                                               <none>       e956badfd97e   2 minutes ago   142MB
<none>                                                               <none>       159cef29cf02   2 minutes ago   143MB
<none>                                                               <none>       0f79de9797c8   2 minutes ago   142MB
<none>                                                               <none>       f04b20055c8c   2 minutes ago   142MB
<none>                                                               <none>       d0fb434ea76d   2 minutes ago   142MB
node                                                                 12-alpine    8dc0ee810c0a   2 days ago      91MB
```

- Copy the Image ID of the 876630244803.dkr.ecr.ap[###].com/sample-nodejs-backend

```
docker rmi --force 6e84f53bb0ea
```

### 2.2 Upload container image to Amazon ECR (frontend)
#### 2.2.1 Create Amazon ECR Repository and Upload Image
- Download the source code to be containerized 
```
cd ~/environment/
```
```
git clone https://github.com/sghaha/sample-react-app.git
``` 

#### 2.2.2 Create an image repository
```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

#### 2.2.3 Bring the authentication token and push the container image to the repository
```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com
```

#### 2.2.4 Build the docker image
```
cd ~/environment/sample-react-app
```
```
npm install
```

```
npm run build
```

```
docker build -t sample-react-app .
```

#### 2.2.5 Enable a docker image tag
```
docker tag sample-react-app:latest $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/sample-react-app:latest
```

#### 2.2.6 Push the image into the repository
```
docker push $ACCOUNT_ID.dkr.ecr.ap-northeast-2.amazonaws.com/sample-react-app:latest
```

#### 2.2.7 Delete the container image
```
docker images -a
```

- Result example시
```
REPOSITORY                                                           TAG          IMAGE ID       CREATED         SIZE
<none>                                                               <none>       85fb9a09b553   2 minutes ago   143MB
<none>                                                               <none>       e8c96d7d909e   2 minutes ago   143MB
876630244803.dkr.ecr.ap[####].com/sample-react-app:latest            latest       6e84f53bb0ea   2 minutes ago   143MB
sample-react-app                                                     latest       6e84f53bb0ea   2 minutes ago   143MB
<none>                                                               <none>       991ab0515040   2 minutes ago   143MB
<none>                                                               <none>       e956badfd97e   2 minutes ago   142MB
<none>                                                               <none>       159cef29cf02   2 minutes ago   143MB
<none>                                                               <none>       0f79de9797c8   2 minutes ago   142MB
<none>                                                               <none>       f04b20055c8c   2 minutes ago   142MB
<none>                                                               <none>       d0fb434ea76d   2 minutes ago   142MB
node                                                                 12-alpine    8dc0ee810c0a   2 days ago      91MB
```

- Copy the Image ID of the 876630244803.dkr.ecr.ap[####].com/sample-react-app:latest

```
docker rmi --force 6e84f53bb0ea
```
