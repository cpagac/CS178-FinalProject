# CS178 Chat App

A cloud-native chat application powered by **Anthropic Claude Haiku**. Built as the CS178 (Cloud Computing and Database Systems) final project at Drake University.

## Live Demo

**Frontend:** http://bedrock-chat-frontend-coleman.s3-website-us-east-1.amazonaws.com

## What It Does

Users open a static web page, type a message, and receive an AI-generated response. The frontend calls a REST API running in Kubernetes; the API forwards the request to the Anthropic API and returns the model's reply.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Static HTML/JS hosted on Amazon S3 |
| Backend | FastAPI (Python) running on Amazon EKS |
| AI Model | Claude Haiku via Anthropic API |
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
  │  Anthropic Python SDK
  ▼
Anthropic API  (Claude Haiku)
```

## Local Development

**Prerequisites:** Python 3.11+, Anthropic API key from [console.anthropic.com](https://console.anthropic.com).

```bash
# Install dependencies
pip install -r backend/requirements.txt

# Start the API
ANTHROPIC_API_KEY=your_key_here uvicorn backend.main:app --reload --port 8000
```

Open `frontend/index.html` in a browser and point it at `http://localhost:8000`.

## Deployment

1. **Install tools** — AWS CLI, `eksctl`, `kubectl`, Docker
2. **Configure AWS** — `aws configure` with an IAM user that has EKS, ECR, and S3 permissions
3. **Create EKS cluster** — `eksctl create cluster --name bedrock-chat --region us-east-1 --nodes 2 --node-type t3.micro`
4. **Build and push image to ECR**
   ```bash
   aws ecr create-repository --repository-name bedrock-chat
   docker buildx build --platform linux/amd64 -t bedrock-chat ./backend
   docker tag bedrock-chat:latest <account>.dkr.ecr.us-east-1.amazonaws.com/bedrock-chat:latest
   docker push <account>.dkr.ecr.us-east-1.amazonaws.com/bedrock-chat:latest
   ```
5. **Create Kubernetes secret** — `kubectl create secret generic anthropic-secret --from-literal=ANTHROPIC_API_KEY=your_key`
6. **Apply Kubernetes manifests** — `kubectl apply -f k8s/`
7. **Host frontend on S3** — create a public bucket with static website hosting enabled, upload `frontend/index.html`, and update `BACKEND_URL` in the file to the EKS LoadBalancer DNS name

## Key AWS Services

- **Amazon S3** — zero-ops static hosting for the frontend
- **Amazon EKS** — managed Kubernetes control plane for the backend
- **Amazon ECR** — private container registry for the Docker image
