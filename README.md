# CS178 Chat App

A cloud-native chat application that streams responses from **Claude 3 Haiku** via AWS Bedrock. Built as the CS178 (Cloud Computing and Database Systems) final project at Drake University.

## What It Does

Users open a static web page, type a message, and receive an AI-generated response. The frontend calls a REST API running in Kubernetes; the API forwards the request to AWS Bedrock and returns the model's reply.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Static HTML/JS hosted on Amazon S3 |
| Backend | FastAPI (Python) running on Amazon EKS |
| AI Model | Claude 3 Haiku via AWS Bedrock |
| Container | Docker, pushed to Amazon ECR |
| Orchestration | Kubernetes (Amazon EKS) |

## Architecture

```
Browser
  │
  ▼
Amazon S3  (static frontend)
  │  HTTP POST /chat
  ▼
EKS LoadBalancer  (AWS Network Load Balancer)
  │
  ▼
FastAPI Pod  (Kubernetes Deployment, 2 replicas)
  │  boto3 InvokeModel API
  ▼
AWS Bedrock  (Claude 3 Haiku — us-east-1)
```

## Local Development

**Prerequisites:** Python 3.11+, AWS credentials configured (`~/.aws/credentials` or environment variables) with Bedrock access in `us-east-1`.

```bash
# Install dependencies
pip install -r backend/requirements.txt

# Start the API
uvicorn backend.main:app --reload --port 8000
```

Open `frontend/index.html` in a browser and point it at `http://localhost:8000`.

## Deployment

1. **Install tools** — AWS CLI, `eksctl`, `kubectl`, Docker
2. **Configure AWS** — `aws configure` with an IAM user that has EKS, ECR, Bedrock, and S3 permissions
3. **Create EKS cluster** — `eksctl create cluster --name bedrock-chat --region us-east-1 --nodes 2`
4. **Build and push image to ECR**
   ```bash
   aws ecr create-repository --repository-name bedrock-chat
   docker build -t bedrock-chat ./backend
   docker tag bedrock-chat:latest <account>.dkr.ecr.us-east-1.amazonaws.com/bedrock-chat:latest
   docker push <account>.dkr.ecr.us-east-1.amazonaws.com/bedrock-chat:latest
   ```
5. **Apply Kubernetes manifests** — `kubectl apply -f k8s/`
6. **Host frontend on S3** — create a public bucket with static website hosting enabled, upload `frontend/index.html`, and update the API URL in the file to the EKS LoadBalancer DNS name

## Key AWS Services

- **Amazon S3** — zero-ops static hosting for the frontend
- **Amazon EKS** — managed Kubernetes control plane for the backend
- **Amazon ECR** — private container registry for the Docker image
- **AWS Bedrock** — serverless access to Claude 3 Haiku; no GPU infrastructure to manage
