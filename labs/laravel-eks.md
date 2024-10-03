# Laravel Application Deployment to AWS EKS

This guide walks you through deploying a Laravel application to **Amazon Elastic Kubernetes Service (EKS)**. It covers setting up the environment, building Docker images, deploying the app, and managing Kubernetes resources.

## Prerequisites

Ensure the following tools are installed and configured:
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [Docker](https://docs.docker.com/get-docker/)

You should also have an AWS account with sufficient permissions to create and manage EKS clusters and ECR repositories.

## Setup Instructions

### 1. Build and Push Docker Images to AWS ECR

#### Set Environment Variables:

```bash
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export REGION=us-west-2    # Choose your AWS region
export REPOSITORY_NAME=my-laravel-app
export IMAGE_TAG=v1.0.0
```
### Create an ECR Repository:

```aws ecr create-repository --repository-name $REPOSITORY_NAME --region $REGION ```

### Authenticate Docker to AWS ECR:
```aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com```

### Build and Push Docker Image:

```
docker-compose build
docker tag IMAGE_NAME:TAG $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPOSITORY_NAME:$IMAGE_TAG
docker push $AWS_ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/$REPOSITORY_NAME:$IMAGE_TAG
```
### 2 Create an EKS Cluster
## Use eksctl to create an EKS cluster:

``` eksctl create cluster --name my-laravel-cluster --region $REGION --nodegroup-name standard-workers --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed ```


Get authentication credentials for the cluster:
``` aws eks --region $REGION update-kubeconfig --name my-laravel-cluster```



### 3. Configure ConfigMaps
Create ConfigMap for Laravel Environment:
``` kubectl create configmap configmap-laravel-env --from-env-file=./docker/app/laravel.dev.env
Apply Nginx Configuration:
kubectl apply -f k8s/configurations/configmap-nginx.yaml
```

### 4. (Optional) Setup HTTPS with TLS Secret
Create a TLS secret:

```
kubectl create secret tls secret-tls --cert [CERTIFICATE_FILE] --key [KEY_FILE]
```

### 5. Deploy the Application
Deploy the Laravel app and the services using Kubernetes manifests:

```
kubectl apply -f k8s/deployments/deployment.yaml
kubectl apply -f k8s/services/service.yaml
```

### 6. Expose the Application via Ingress
Ensure the AWS Load Balancer Controller is installed (follow the guide).

Then, apply the Ingress resource:

```
kubectl apply -f k8s/ingress/ingress.yaml 
```

### 7. Verify the Deployment
Once deployed, get the Load Balancer DNS name:

```
kubectl get svc
```
### To verify the application:

```
curl -H "host: DOMAIN_NAME" http://LOAD_BALANCER_DNS
```

## Troubleshooting
Force Pod Update
If changes to the image are not picked up, update the imagePullPolicy in your deployment file to:

```
imagePullPolicy: "Always" 
```
### Then reapply the deployment:

```
kubectl apply -f k8s/deployments/deployment.yaml 
```

### Additional Resources

Database: Use Amazon RDS for a managed database service.
Ingress: AWS EKS uses the AWS load balancer controller to manage ingress resources.
Security: Secure your EKS cluster using IAM roles for service accounts and network policies.
License
This project is licensed under the MIT License. See the LICENSE file for details.

