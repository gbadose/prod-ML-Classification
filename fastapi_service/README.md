# FastAPI Model Service on ECS

This folder contains a minimal FastAPI application that serves the dog breed classifier model. The service expects a PyTorch model file named `model.pth` to be available inside the container. You can mount or copy the model when building the image.

## Building the Docker image

```bash
# copy your trained model into this directory
docker build -t dog-classifier-fastapi .
```

## Pushing to Amazon ECR

```bash
aws ecr create-repository --repository-name dog-classifier-fastapi
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION=us-east-1
aws ecr get-login-password --region $REGION | \
  docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com

docker tag dog-classifier-fastapi:latest $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/dog-classifier-fastapi:latest
docker push $ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/dog-classifier-fastapi:latest
```

## Deploying on ECS with Auto Scaling

The example below creates a Fargate service with an application load balancer and configures target tracking scaling based on CPU utilization.

```bash
# Create a cluster
aws ecs create-cluster --cluster-name dog-classifier-cluster

# Register the task definition
aws ecs register-task-definition \
  --family dog-classifier-task \
  --requires-compatibilities FARGATE \
  --network-mode awsvpc \
  --cpu "512" --memory "1024" \
  --execution-role-arn <ecsTaskExecutionRoleArn> \
  --container-definitions file://container_definitions.json

# Create service with an application load balancer
aws ecs create-service \
  --cluster dog-classifier-cluster \
  --service-name dog-classifier-service \
  --task-definition dog-classifier-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxx],securityGroups=[sg-xxxx],assignPublicIp=ENABLED}" \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/...,containerName=dog-classifier,containerPort=8080"

# Configure auto scaling
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/dog-classifier-cluster/dog-classifier-service \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 1 --max-capacity 4

aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id service/dog-classifier-cluster/dog-classifier-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{"TargetValue":50.0,"PredefinedMetricSpecification":{"PredefinedMetricType":"ECSServiceAverageCPUUtilization"}}'
```

These commands assume that networking resources such as VPC, subnets, and security groups already exist. Update the placeholders with your specific resource identifiers.
