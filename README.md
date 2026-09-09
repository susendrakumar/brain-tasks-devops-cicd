# Brain Tasks App – DevOps CI/CD Deployment

## Project Overview

This project demonstrates an end-to-end DevOps CI/CD deployment workflow for a provided web application.

The application was provided as a pre-built `dist` directory containing compiled HTML, CSS, JavaScript, and static assets. The objective of this assignment was to containerize the application, store the Docker image in a container registry, deploy it to Kubernetes on Amazon EKS, and automate the build and deployment process using AWS CodeBuild and AWS CodePipeline.

The application was:

- Containerized using Docker and Nginx
- Tested locally on port `3000`
- Stored in Amazon Elastic Container Registry (ECR)
- Deployed to Amazon Elastic Kubernetes Service (EKS)
- Managed using Kubernetes
- Exposed publicly using an AWS Load Balancer
- Integrated with AWS CodeBuild and AWS CodePipeline
- Monitored using Amazon CloudWatch Logs

---

## DevOps Architecture

```text
GitHub
   ↓
AWS CodePipeline
   ↓
AWS CodeBuild
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes Deployment
   ↓
AWS Load Balancer
   ↓
Live Application
```

AWS CodeBuild performs the EKS deployment using `kubectl` commands defined in `buildspec.yml`.

---

## Technologies Used

- Git
- GitHub
- Docker
- Nginx
- Amazon ECR
- Amazon EKS
- Kubernetes
- AWS CodeBuild
- AWS CodePipeline
- Amazon CloudWatch
- AWS IAM
- AWS Load Balancer
- AWS CLI
- kubectl

---

## Project Files

The repository contains the following important files:

```text
dist/
Dockerfile
buildspec.yml
deployment.yaml
service.yaml
README.md
```

### File Purpose

- `dist/` – Provided production-ready application files
- `Dockerfile` – Builds the Nginx-based application container
- `buildspec.yml` – Defines AWS CodeBuild build and deployment commands
- `deployment.yaml` – Defines the Kubernetes Deployment
- `service.yaml` – Defines the Kubernetes LoadBalancer Service
- `README.md` – Project setup, architecture, deployment process, and evidence

---

# 1. Application and Docker Containerization

The provided application was containerized using Docker.

Nginx was used to serve the production-ready files from the `dist` directory.

The Docker image was built using:

```bash
docker build -t brain-tasks-app .
```

The Docker container was tested locally using:

```bash
docker run -d --name brain-tasks-container -p 3000:80 brain-tasks-app
```

The application was successfully verified locally using:

```text
http://localhost:3000
```

This confirmed that the Docker image could successfully serve the application before deploying it to AWS.

---

# 2. Version Control – GitHub

Git and GitHub were used for source control.

The application code and DevOps configuration files were maintained in the GitHub repository and pushed to the `main` branch using Git CLI commands.

Example workflow:

```bash
git add .
git commit -m "Update DevOps deployment"
git push origin main
```

GitHub acts as the source for the CI/CD pipeline.

---

# 3. Amazon ECR – Container Registry

Amazon Elastic Container Registry (ECR) was used to store Docker images.

### ECR Repository

```text
brain-tasks-app
```

### Repository URI

```text
629845796500.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app
```

The Docker image was built, tagged, authenticated, and pushed to Amazon ECR.

The `latest` tag was maintained along with commit-specific image tags generated during the CI/CD process.

Amazon EKS retrieves the application container image from this ECR repository during deployment.

---

# 4. Amazon EKS – Kubernetes Cluster

Amazon Elastic Kubernetes Service (EKS) was used to host the Kubernetes workload.

### Cluster Name

```text
brain-tasks-cluster
```

### AWS Region

```text
ap-south-1 (Mumbai)
```

The EKS cluster was configured with worker capacity for running the application workload.

The Kubernetes configuration was connected to the EKS cluster using AWS CLI.

Example:

```bash
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster
```

The cluster connection was verified using `kubectl`.

---

# 5. Kubernetes Deployment

The Kubernetes deployment is defined using:

```text
deployment.yaml
service.yaml
```

The resources can be deployed using:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

The deployment was verified using:

```bash
kubectl get pods
kubectl get svc
```

The application pod successfully reached:

```text
READY: 1/1
STATUS: Running
RESTARTS: 0
```

The Kubernetes Service was configured as:

```text
Type: LoadBalancer
```

This automatically provisioned an AWS Load Balancer for public access to the application.

---

# 6. AWS CodeBuild

AWS CodeBuild was configured to automate the Docker build, ECR push, and EKS deployment process.

### CodeBuild Project

```text
brain-tasks-codebuild
```

The project uses an AWS-managed build environment.

The `buildspec.yml` performs the following operations:

1. Installs/configures `kubectl`
2. Authenticates Docker with Amazon ECR
3. Generates an image tag from the source revision
4. Builds the Docker image
5. Tags the image
6. Pushes the Docker image to Amazon ECR
7. Updates the local kubeconfig for the EKS cluster
8. Applies the Kubernetes Deployment
9. Applies the Kubernetes LoadBalancer Service
10. Updates the Kubernetes deployment with the newly built image
11. Waits for the deployment rollout to complete

The deployment is verified using:

```bash
kubectl rollout status deployment/brain-tasks-deployment --timeout=180s
```

Required IAM permissions were configured to allow CodeBuild to interact with Amazon ECR and Amazon EKS.

---

# 7. AWS CodePipeline – CI/CD Automation

AWS CodePipeline was configured to automate the CI/CD workflow.

### Pipeline Name

```text
brain-tasks-pipeline
```

### Pipeline Flow

```text
GitHub Source
      ↓
AWS CodePipeline
      ↓
AWS CodeBuild
      ↓
Docker Build
      ↓
Amazon ECR
      ↓
kubectl Deployment
      ↓
Amazon EKS
      ↓
Kubernetes
      ↓
AWS Load Balancer
```

GitHub is configured as the source and AWS CodeBuild performs the build and Kubernetes deployment operations.

A source change can trigger the pipeline, which then invokes CodeBuild to build and push the Docker image and deploy the updated image to Amazon EKS using `kubectl`.

---

# 8. CI/CD Verification

The CI/CD workflow was tested using GitHub commits.

The verified deployment process was:

1. Code was committed and pushed to GitHub
2. AWS CodePipeline detected the source change
3. The Source stage completed
4. AWS CodeBuild started automatically
5. Docker built the application image
6. The Docker image was pushed to Amazon ECR
7. CodeBuild connected to the Amazon EKS cluster
8. Kubernetes resources were applied/updated
9. The Kubernetes deployment rollout completed
10. The application pod reached the `Running` state
11. The application was accessible through the AWS Load Balancer

This verified the end-to-end CI/CD deployment workflow.

---

# 9. Monitoring – Amazon CloudWatch

Amazon CloudWatch Logs was used to monitor the build and deployment process.

AWS CodeBuild sends its build and deployment logs to the following CloudWatch Log Group:

```text
/aws/codebuild/brain-tasks-codebuild
```

CloudWatch Logs were used to monitor:

- CodeBuild execution
- Docker build and push operations
- EKS deployment commands
- Kubernetes deployment updates
- Kubernetes rollout status
- Build phase completion

The deployment logs confirmed:

```text
brain-tasks-deployment image updated
successfully rolled out
Deployment completed successfully
POST_BUILD State: SUCCEEDED
```

This provides monitoring evidence for the automated build and deployment workflow.

---

# 10. Kubernetes Load Balancer

The application is publicly exposed through a Kubernetes Service of type:

```text
LoadBalancer
```

The Kubernetes service automatically provisioned an AWS Classic Load Balancer.

### Load Balancer Details

```text
Load Balancer Name:
ae85fa46c3315433a837c834d26d8dc4

Type:
Classic

Scheme:
Internet-facing

Listener:
TCP:80

Kubernetes Service:
brain-tasks-service
```

The Load Balancer showed healthy backend instances and successfully routed traffic to the Kubernetes application.

The generated AWS Load Balancer DNS endpoint was used to access and verify the deployed Brain Tasks application.

> Note: The Load Balancer endpoint is dynamically provisioned by Kubernetes and may no longer be available after AWS resources are cleaned up.

---

# 11. Deployment Evidence

The following screenshots are included in this repository as deployment evidence:

| Screenshot | Verification |
|---|---|
| `01_ECR_Docker_Image.png` | Docker image stored in Amazon ECR |
| `02_EKS_Cluster_Active.png` | Amazon EKS cluster in Active state |
| `03_Kubernetes_Pod_Running.png` | Kubernetes application pod running successfully |
| `04_Kubernetes_LoadBalancer_Service.png` | Kubernetes LoadBalancer Service |
| `05_Live_Application_AWS_LoadBalancer.png` | Application accessible through AWS Load Balancer |
| `06_AWS_CodeBuild_Succeeded.png` | Successful AWS CodeBuild executions |
| `07_AWS_CodePipeline_CICD_Succeeded.png` | Successful CodePipeline Source and Build stages |
| `08_Automatic_GitHub_CICD_Trigger_Succeeded.png` | GitHub source change triggering the CI/CD workflow |
| `09_CloudWatch_Deployment_Logs.png` | CloudWatch build and deployment logs |
| `10_Kubernetes_LoadBalancer_Details.png` | AWS Load Balancer deployment details |

---

# Final Result

The provided Brain Tasks application was successfully:

- Containerized using Docker and Nginx
- Tested locally on port `3000`
- Version controlled using Git and GitHub
- Stored as a Docker image in Amazon ECR
- Deployed to Amazon EKS
- Managed using Kubernetes
- Exposed publicly using an AWS Load Balancer
- Built automatically using AWS CodeBuild
- Integrated with AWS CodePipeline
- Deployed to EKS using automated `kubectl` commands
- Monitored using Amazon CloudWatch Logs
- Verified using deployment screenshots

## Final Deployment Flow

```text
GitHub Push
     ↓
AWS CodePipeline
     ↓
AWS CodeBuild
     ↓
Docker Image Build
     ↓
Amazon ECR
     ↓
Amazon EKS
     ↓
Kubernetes Deployment
     ↓
AWS Load Balancer
     ↓
Live Application
```

The end-to-end DevOps CI/CD deployment workflow was successfully completed.
