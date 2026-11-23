# AI Aircraft Maintenance Platform

![AI Aircraft Maintenance Platform](https://github.com/user-attachments/assets/47427565-461c-49eb-84f6-d8997e078d4c)

An end-to-end aircraft maintenance decision-support platform that combines:

- React-based operational dashboard
- FastAPI engineering analytics service
- Deterministic Excel telemetry analysis
- Amazon Bedrock generative AI analysis
- AWS ECR container registry
- Amazon EKS Kubernetes deployment
- GitHub Actions CI/CD automation

The application accepts aircraft telemetry in Excel format, calculates a structured engineering summary, compares the result with an internal maintenance manual using Amazon Bedrock, and presents maintenance recommendations through a dashboard.

> **Safety notice:** This project is a decision-support prototype, not a certified aircraft maintenance system. Generated recommendations must be reviewed by qualified aviation engineering and maintenance personnel.

---

## Table of Contents

- [What the Project Does](#what-the-project-does)
- [Application Workflow](#application-workflow)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Core Features](#core-features)
- [Local Development](#local-development)
- [API Reference](#api-reference)
- [Docker Containerization](#docker-containerization)
- [AWS and Amazon EKS Setup](#aws-and-amazon-eks-setup)
- [DevOps and CI/CD Implementation](#devops-and-cicd-implementation)
- [GitHub Actions Configuration](#github-actions-configuration)
- [Deployment and Verification](#deployment-and-verification)
- [Current Configuration Notes](#current-configuration-notes)
- [Security and Production Recommendations](#security-and-production-recommendations)
- [Troubleshooting](#troubleshooting)
- [Documentation](#documentation)

---

## What the Project Does

The platform provides a three-stage maintenance assessment:

1. **Engineering analytics**
   - Reads an uploaded `.xlsx` or `.xls` aircraft telemetry file.
   - Normalizes common Excel column names.
   - Selects the first available aircraft ID.
   - Uses the latest flight cycle as the current record.
   - Compares the latest sensor values with the previous 10 flight records.
   - Calculates averages, medians, standard deviations, absolute changes, and percentage changes.

2. **AI maintenance analysis**
   - Sends the deterministic analytics JSON and the bundled AeroTech ATX200 maintenance manual PDF to Amazon Bedrock.
   - Uses Amazon Nova Pro through the Bedrock Runtime `Converse` API.
   - Requests a structured JSON maintenance report grounded in the manual.

3. **Operational dashboard**
   - Displays aircraft and engine telemetry.
   - Highlights risk, remaining useful life, vibration, and flight-cycle metrics.
   - Shows health status, threshold violations, root-cause analysis, maintenance actions, inspection checklists, and work-order information.
   - Presents a clear fly/no-fly decision for dashboard review.

The AI step does not replace deterministic analytics. It receives measured analytics as trusted context and uses the maintenance manual as the source for procedural guidance.

---

## Application Workflow

```text
Aircraft telemetry Excel file
            |
            v
React dashboard
            |
            v
FastAPI /aircraft/analytics
            |
            v
Pandas data cleaning and historical analysis
            |
            v
Engineering summary JSON
            |
            v
FastAPI /aircraft/maintenance-prediction
            |
            +----------------------+
            |                      |
            v                      v
    Maintenance manual PDF    Engineering JSON
            |                      |
            +----------+-----------+
                       |
                       v
      Amazon Bedrock Nova Pro
                       |
                       v
          Structured maintenance report
                       |
                       v
             React dashboard
```

---

## Architecture

```text
Developer
   |
   | git push main
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   +--> Build backend Docker image
   +--> Build frontend Docker image
   +--> Push images to Amazon ECR
   |
   v
Configure kubectl for Amazon EKS
   |
   v
Replace image placeholders in Kubernetes manifests
   |
   v
Create/update AWS credentials Kubernetes Secret
   |
   v
kubectl apply -f kubernetes/
   |
   +--> Backend Deployment + LoadBalancer Service
   +--> Frontend Deployment + LoadBalancer Service
   |
   v
Amazon EKS Worker Nodes
   |
   +--> FastAPI pods
   +--> React/Vite pods
   |
   v
Amazon Bedrock Runtime
```

The backend and frontend are deployed independently so that application images can be versioned and rolled out separately.

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, JavaScript |
| Backend API | Python 3.12, FastAPI, Uvicorn |
| Data Analytics | Pandas, OpenPyXL |
| AI / LLM | Amazon Bedrock, Nova Pro, Bedrock Converse API |
| AWS SDK | Boto3 |
| Containers | Docker |
| Orchestration | Kubernetes / Amazon EKS |
| Container Registry | Amazon Elastic Container Registry (ECR) |
| CI/CD | GitHub Actions |
| Cloud Platform | Amazon Web Services |

---

## Repository Structure

```text
AI-AIRCRAFT-MAINTENANCE-PLATFORM/
|
|-- .github/
|   `-- workflows/
|       `-- deploy.yml             # Main GitHub Actions CI/CD pipeline
|
|-- docker-compose.yml             # Local frontend + backend orchestration
|-- backend/
|   |-- app.py                     # FastAPI endpoints
|   |-- dockerfile                 # Backend container definition
|   |-- requirements.txt
|   |-- pyproject.toml
|   |-- src/
|   |   `-- aircraft_maintenance/
|   |       |-- engineering_analytics.py
|   |       `-- bedrock_maintenance_analyzer.py
|   `-- data/
|       |-- aircraft_maintenance_intelligence_dataset.xlsx
|       `-- AeroTech_ATX200_Maintenance_Manual.pdf
|
|-- frontend/
|   |-- dockerfile                 # Frontend container definition
|   |-- package.json
|   |-- src/
|   |   |-- App.jsx
|   |   |-- api/client.js
|   |   `-- components/
|   `-- .env.example
|
|-- kubernetes/
|   |-- backend-deployment.yaml
|   |-- backend-service.yaml
|   |-- frontend-deployment.yaml
|   `-- frontend-service.yaml
|
|-- Github-Actions-CICD.md
|-- Deploying_AI_Aircraft_Maintenance_Application_on_Amazon_EKS_Imperative.md
`-- README.md
```

---

## Core Features

### Engineering Analytics

- Excel dataset loading and validation
- Header normalization for units and aliases
- Aircraft list extraction
- Latest-record selection
- 10-record historical window
- Sensor trend analysis
- Historical average, median, standard deviation, and percent change
- JSON-safe analytical output

### AI-Powered Maintenance Report

The Bedrock prompt requests:

- Aircraft and aircraft model
- Health status
- Risk level
- Threshold violations
- Root-cause analysis
- Maintenance recommendations
- Inspection checklist
- Flight readiness
- Final fly/no-fly decision
- Generated work order
- Confidence and missing-information fields

The prompt explicitly asks the model to use the supplied maintenance manual and to state when a threshold or procedure is unavailable instead of inventing it.

### Dashboard

- Backend health status
- Excel file upload
- Engineering metrics
- Engine gauges
- Historical signal trends
- AI-generated maintenance report
- Health and risk badges
- Inspection and work-order presentation

---

## Local Development

### Prerequisites

- Python 3.12+
- Node.js 20+
- npm
- AWS CLI
- Docker
- Access to an AWS account with Bedrock enabled
- Bedrock model access for `amazon.nova-pro-v1:0`

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd AI-AIRCRAFT-MAINTENANCE-PLATFORM
```

### 2. Configure the Backend

Create a backend `.env` file or export environment variables:

```env
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=<aws-access-key-id>
AWS_SECRET_ACCESS_KEY=<aws-secret-access-key>
BEDROCK_MODEL_ID=amazon.nova-pro-v1:0
```

For local development, AWS CLI credentials can also be used because Boto3 falls back to the standard AWS credential provider chain when explicit credentials are not supplied.

### 3. Start the Backend

```bash
cd backend
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate

pip install -r requirements.txt
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

FastAPI documentation will be available at:

- Swagger UI: `http://localhost:8000/docs`
- Health endpoint: `http://localhost:8000/testing/health`

### 4. Start the Frontend

```bash
cd frontend
npm install
npm run dev
```

By default, Vite serves the frontend on `http://localhost:5173`.

The API client reads `VITE_API_BASE_URL`; it defaults to `http://localhost:8000`. Set this variable when using a different backend.

### Run the Full Stack with Docker Compose

From the repository root, optionally provide AWS credentials in your shell or a root `.env` file (do not commit credentials), then run:

```bash
docker compose up --build
```

Open the frontend at `http://localhost:5173` and the API docs at `http://localhost:8000/docs`. To stop the containers, press `Ctrl+C`, then run `docker compose down`.

The Compose setup starts both services, maps their ports to the host, points the frontend at the host-accessible backend URL, and passes AWS settings to the backend for Bedrock requests. Excel analytics and the health endpoint can run without AWS credentials; Bedrock-backed recommendations require valid AWS credentials and model access.

---

## API Reference

### `GET /`

Basic service status.

```json
{
  "status": "ok",
  "service": "aircraft-maintenance-api",
  "message": "Backend is running"
}
```

### `GET /testing/health`

Health endpoint for smoke tests.

```json
{
  "status": "ok",
  "service": "aircraft-maintenance-api"
}
```

### `POST /aircraft/analytics`

Uploads an Excel file as `multipart/form-data`.

**Field name:** `excel_file`

**Response:**

```json
{
  "message": "Analytics generated successfully",
  "aircraft_id": "AIRCRAFT-001",
  "summary": {
    "aircraft_id": "AIRCRAFT-001",
    "latest_flight_cycle": 100,
    "current_record": {},
    "historical_window_size": 10,
    "historical_analysis": []
  }
}
```

The analytics endpoint:

- Uses the first sheet by default.
- Selects the first aircraft returned from the uploaded dataset.
- Uses a 10-cycle historical window.

### `POST /aircraft/maintenance-prediction`

Sends the complete analytics response or the nested `summary` object for AI analysis.

Example request body:

```json
{
  "summary": {
    "aircraft_id": "AIRCRAFT-001",
    "latest_flight_cycle": 100,
    "current_record": {},
    "historical_window_size": 10,
    "historical_analysis": []
  }
}
```

The response contains a `success` flag and a structured `report` object.

---

## Docker Containerization

The root `docker-compose.yml` runs the frontend and backend together for local development.

### Build Images Locally

```bash
docker build -t maintenance-backend ./backend
docker build -t maintenance-frontend ./frontend
```

### Run the Backend

```bash
docker run --rm -p 8000:8000 \
  -e AWS_REGION=us-east-1 \
  -e AWS_ACCESS_KEY_ID=<key> \
  -e AWS_SECRET_ACCESS_KEY=<secret> \
  -e BEDROCK_MODEL_ID=amazon.nova-pro-v1:0 \
  maintenance-backend
```

### Run the Frontend

```bash
docker run --rm -p 3000:3000 maintenance-frontend
```

The backend Docker image uses Python 3.12 slim, installs dependencies with `uv`, copies the maintenance data and PDF, and starts Uvicorn on port `8000`.

The frontend image uses Node 20 Alpine and starts Vite through npm.

---

## AWS and Amazon EKS Setup

### Required AWS Resources

- EKS cluster
- Managed node group
- Amazon ECR repositories:
  - `maintenance-backend`
  - `maintenance-frontend`
- IAM principal or role allowed to deploy to EKS
- Bedrock access for the configured inference profile/model

### Recommended Worker Node Configuration

The project guide uses:

- Amazon Linux 2023
- `t3.medium` nodes
- Desired size: 2
- Minimum size: 1
- Maximum size: 2

The worker node IAM role should have at least:

- `AmazonEKSWorkerNodePolicy`
- `AmazonEKS_CNI_Policy`
- `AmazonEC2ContainerRegistryReadOnly`

### Create ECR Repositories

```bash
aws ecr create-repository \
  --repository-name maintenance-backend \
  --region us-east-1

aws ecr create-repository \
  --repository-name maintenance-frontend \
  --region us-east-1
```

### Connect kubectl

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name <EKS_CLUSTER_NAME>

kubectl config current-context
kubectl get nodes
```

The IAM identity used by kubectl must have EKS cluster access, typically through an EKS Access Entry and `AmazonEKSClusterAdminPolicy`.

---

## DevOps and CI/CD Implementation

DevOps was implemented using Docker, Amazon ECR, Kubernetes, Amazon EKS, and GitHub Actions.

### 1. Containerization

Two Docker images create reproducible runtime environments:

- `maintenance-backend`: FastAPI, analytics dependencies, Boto3, and maintenance data
- `maintenance-frontend`: React and Vite application

Dependencies are installed inside the images, avoiding reliance on a developer machine.

### 2. Container Registry

GitHub Actions authenticates with AWS and Amazon ECR, tags both images, and pushes them to the configured ECR repositories.

### 3. Kubernetes Deployment

The `kubernetes/` directory describes:

- Two `Deployment` resources
- One pod replica per application
- One `LoadBalancer` `Service` for the frontend
- One `LoadBalancer` `Service` for the backend
- Container ports `8000` and `3000`

The Kubernetes Deployments use `imagePullPolicy: Always`, ensuring nodes attempt to pull the latest image URI.

### 4. Secrets Management

GitHub Actions reads AWS credentials and image values from repository secrets.

The workflow then creates or updates a Kubernetes Secret named `aws-credentials` and attaches it to the backend pod through `envFrom.secretRef`.

### 5. Automated Deployment

Every push to `main` triggers the deployment workflow. GitHub Actions:

1. Checks out the source code.
2. Configures temporary AWS credentials.
3. Logs in to Amazon ECR.
4. Builds and pushes the backend image.
5. Builds and pushes the frontend image.
6. Configures kubectl for the EKS cluster.
7. Replaces backend and frontend image placeholders.
8. Creates or updates the Kubernetes AWS Secret.
9. Applies all Kubernetes manifests.
10. Waits for both deployments to complete rollout.
11. Lists deployments, pods, and services.

### 6. Deployment Verification

The pipeline verifies health through Kubernetes rollout status:

```bash
kubectl rollout status deployment/maintenance-backend
kubectl rollout status deployment/maintenance-frontend
```

It also prints:

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

---

## GitHub Actions Configuration

The workflow is located at [deploy.yml](.github/workflows/deploy.yml).

### Workflow Trigger

The pipeline runs only on pushes to the `main` branch.

### Required GitHub Repository Secrets

Open:

```text
Repository > Settings > Secrets and variables > Actions
```

| Secret | Purpose |
|---|---|
| `AWS_ACCESS_KEY_ID` | AWS authentication for GitHub Actions |
| `AWS_SECRET_ACCESS_KEY` | AWS authentication for GitHub Actions |
| `AWS_REGION` | AWS region, for example `us-east-1` |
| `AWS_ACCOUNT_ID` | AWS account identifier |
| `EKS_CLUSTER_NAME` | Target Amazon EKS cluster |
| `BACKEND_IMAGE` | Full backend ECR image URI |
| `FRONTEND_IMAGE` | Full frontend ECR image URI |

Example image values:

```text
<account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-backend:latest
<account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-frontend:latest
```

### Trigger a Deployment

```bash
git add .
git commit -m "Update AI maintenance platform"
git push origin main
```

The workflow run will appear under the repository's **Actions** tab.

---

## Deployment and Verification

### Apply Manually

```bash
sed -i "s|\${BACKEND_IMAGE}|<BACKEND_ECR_IMAGE_URI>|g" kubernetes/backend-deployment.yaml
sed -i "s|\${FRONTEND_IMAGE}|<FRONTEND_ECR_IMAGE_URI>|g" kubernetes/frontend-deployment.yaml

kubectl create secret generic aws-credentials \
  --from-literal=AWS_ACCESS_KEY_ID=<key> \
  --from-literal=AWS_SECRET_ACCESS_KEY=<secret> \
  --from-literal=AWS_REGION=us-east-1 \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl apply -f kubernetes/
```

### Check Resources

```bash
kubectl get deployments
kubectl get pods
kubectl get services
kubectl get svc
```

### Find LoadBalancer URLs

```bash
kubectl get svc maintenance-backend
kubectl get svc maintenance-frontend
```

After AWS has provisioned both load balancers, use the external DNS names to open the frontend and test the backend health endpoint.

### Update the Frontend Backend URL

The frontend API client currently uses a fixed deployed backend LoadBalancer URL:

```javascript
const API_BASE_URL = 'http://<backend-load-balancer-dns>';
```

After creating a new backend LoadBalancer, replace this value or update the client to read `VITE_API_BASE_URL` and rebuild the frontend image.

---

## Current Configuration Notes

The following items describe the current implementation and should be considered when operating or improving the project:

1. **Image tagging:** Images are tagged with `latest`. Commit-SHA or semantic version tags are recommended for immutable rollbacks.
2. **Frontend port:** The Kubernetes frontend service targets port `3000`, while the current Vite Docker command should explicitly serve Vite on port `3000` for the manifest to match reliably.
3. **Backend URL:** `frontend/src/api/client.js` contains a fixed LoadBalancer URL rather than reading an environment variable.
4. **Replica count:** Both deployments use one replica.
5. **AWS credentials:** The workflow stores AWS credentials in a Kubernetes Secret. IRSA or workload identity is preferable for production.
6. **Load balancers:** Both frontend and backend use Kubernetes `LoadBalancer` Services. An Ingress controller is a better long-term entry point for routing and TLS.
7. **No health probes:** Kubernetes Deployments do not currently contain startup, readiness, or liveness probes.
8. **No tests or lint jobs:** The current pipeline does not run unit tests, backend linting, frontend linting, or container vulnerability scans.
9. **AI output:** The model output is parsed as JSON, but all generated maintenance decisions still require human and operational validation.

---

## Security and Production Recommendations

For a production-ready implementation:

- Replace GitHub static AWS access keys with GitHub OIDC and short-lived AWS credentials.
- Use IAM Roles for Service Accounts (IRSA) for Bedrock access.
- Remove AWS credentials from Kubernetes manifests and pod environment variables.
- Store sensitive configuration in AWS Secrets Manager or a managed external secret controller.
- Use private ECR repositories and least-privilege pull permissions.
- Use commit-SHA image tags.
- Add readiness and liveness probes.
- Add request limits and upload-size validation.
- Add frontend and backend test suites.
- Add SonarQube, ESLint, Ruff, or equivalent quality checks.
- Add image scanning and dependency scanning.
- Pin or digest-pin dependencies and base images.
- Use an Ingress, AWS Load Balancer Controller, and TLS certificates.
- Add network policies and Kubernetes resource requests/limits.
- Add horizontal pod autoscaling.
- Restrict CORS from `*` to approved frontend origins.
- Add authentication and role-based authorization.
- Add audit logging for uploaded files and AI decisions.
- Use infrastructure-as-code for EKS, VPC, IAM, ECR, and monitoring.
- Consider Helm or Kustomize to manage environment-specific manifests.
- Add CloudWatch, Prometheus, or OpenTelemetry observability.
- Add a formal rollback and incident-response process.

---

## Troubleshooting

### Backend Cannot Invoke Bedrock

Verify:

- Bedrock is available in the configured region.
- Nova Pro model access is enabled.
- The pod or local environment has valid AWS credentials.
- The IAM identity has `bedrock:InvokeModel` permission.
- `BEDROCK_MODEL_ID` is valid.

### Frontend Cannot Reach the Backend

Verify:

- The backend LoadBalancer external address exists.
- `frontend/src/api/client.js` contains the correct backend URL.
- CORS permits the frontend origin.
- Port `8000` is reachable for the backend service.
- The frontend service target port matches the running Vite server port.

### Pod Is in `ImagePullBackOff`

Verify:

- ECR repository names and URIs are correct.
- The image was pushed successfully.
- Worker nodes can authenticate to ECR.
- The IAM node role has ECR pull permissions.

### Pod Does Not Start

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Kubernetes Authentication Fails

Check the IAM Access Entry and ensure the principal is mapped to `AmazonEKSClusterAdminPolicy`. Then refresh kubeconfig:

```bash
aws eks update-kubeconfig --region us-east-1 --name <EKS_CLUSTER_NAME>
```

### `kubectl rollout status` Times Out

Inspect pod events and logs:

```bash
kubectl get pods
kubectl describe deployment maintenance-backend
kubectl describe deployment maintenance-frontend
kubectl logs deployment/maintenance-backend
kubectl logs deployment/maintenance-frontend
```

---

## Documentation

- [GitHub Actions CI/CD Guide](Github-Actions-CICD.md)
- [Amazon EKS Imperative Deployment Guide](Deploying_AI_Aircraft_Maintenance_Application_on_Amazon_EKS_Imperative.md)
- [FastAPI API documentation](http://localhost:8000/docs) — available when the local backend is running

---

## License

See [LICENSE](LICENSE).
