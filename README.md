# prod-ML-Classification
In this project, I ensure to:
1. Train and deploy a model on Sagemaker, using the most appropriate instances. Set up multi-instance training in my Sagemaker notebook.
2. Adjust my Sagemaker notebooks to perform training and deployment on EC2.
3. Set up a Lambda function for my deployed model. Set up auto-scaling for my deployed endpoint as well as concurrency for my Lambda function.
4. Ensure that the security on my ML pipeline is set up properly.

## FastAPI Service Deployment on ECS

See [fastapi_service/README.md](fastapi_service/README.md) for instructions on containerizing the model inference API with FastAPI and deploying it to Amazon ECS with auto-scaling.
