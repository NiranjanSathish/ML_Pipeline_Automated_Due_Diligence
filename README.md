# Market Due Diligence AI Agent (Agentic RAG) 

A production-grade, autonomous multi-agent system for financial analysis, powered by **LangGraph**, **FastAPI**, **Streamlit**, and **Google Vertex AI**.

## Deployment Automation & CI/CD

This project features a fully automated **CI/CD Pipeline** using **GitHub Actions**, removing the need for manual deployments.

### The Deployment Pipeline (`.github/workflows/ci-cd.yaml`)
Every push to any branch triggers the following sequential pipeline:

#### **Stage 1: Continuous Integration (CI)**
- **Syntax Checks**: Runs `flake8` to ensure code quality.
- **Unit Tests**: Runs `pytest` to verify logic.
- **Model Validation & Bias Check**:
  - Runs the **Evaluator Agent** against a Golden Dataset.
  - **Bias Threshold**: The pipeline **FAILS** if the Global Quality Score is below **0.2**.
  - **Metrics**: Checks Factual Accuracy, Groundedness, Answer Relevancy, and Soft Recall.

#### **Stage 2: Continuous Deployment (CD) - Backend API**
- **Trigger**: Automatic (only if Stage 1 passes).
- **Build**: Builds parameters-optimized Docker Image (`agent-api`).
- **Push**: Pushes image to **Google Artifact Registry (GAR)**.
- **Deploy**: Updates **Google Kubernetes Engine (GKE)** cluster `agent-cluster`.
- **Zero-Downtime Rollback**: If the new deployment fails health checks, it **automatically rolls back** to the previous stable version.

#### **Stage 3: Continuous Deployment (CD) - Frontend UI**
- **Trigger**: Automatic (only if Stage 1 passes).
- **Build**: Builds Streamlit UI Image (`agent-ui`).
- **Deploy**: Deploys to **Google Cloud Run** as a serverless service.
- **Access**: Publicly accessible via the Cloud Run URL.

#### **Stage 4: Notification**
- **Email Alert**: Sends a confirmation email with deployment details upon success.

---

## Architecture & Tech Stack

- **Orchestration**: LangGraph (Stateful Multi-Agent Graph)
- **Agents**:
  - **Planner**: Deconstructs complex queries.
  - **Researcher**: Fetches data from Qdrant & External Tools.
  - **Synthesizer**: Compiles final reports.
  - **Evaluator**: Validates output quality.
- **Vector DB**: Qdrant (Hybrid Search).
- **LLM**: Google Vertex AI (Gemini Pro).
- **Backend**: FastAPI.
- **Frontend**: Streamlit.
- **Infrastructure**: GKE (API), Cloud Run (UI), GAR (Registry).

---

## Setup & Local Development

### 1. Prerequisites
- Python 3.10+
- Docker
- Google Cloud SDK

### 2. Environment Configuration
Create a `.env` file in the root directory:
```properties
# Google Cloud
GCP_PROJECT_ID=coherent-rite-473622-j0
GCP_LOCATION=us-central1
GOOGLE_APPLICATION_CREDENTIALS=vertex-key.json

# Vector DB
QDRANT_URL=https://your-qdrant-url
QDRANT_API_KEY=your-api-key
QDRANT_COLLECTION_NAME=financial_data

# LLMFlow
OPENAI_API_KEY=sk-... (If used)
```

### 3. Running Locally
You can run the full stack using Docker directly:
```bash
# Backend (Port 8080)
docker run -p 8080:8080 --env-file .env us-central1-docker.pkg.dev/coherent-rite-473622-j0/agents-repo/agent-api:v2

# Frontend (Port 8501)
docker run -p 8501:8501 us-central1-docker.pkg.dev/coherent-rite-473622-j0/agents-repo/agent-ui:v1
```

---

## GitHub Secrets Configuration

For the **CI/CD Pipeline** to work, you must configure the following **Secrets** in your GitHub Repository:

| Secret Name | Description |
|---|---|
| `GCP_SA_KEY` | Service Account JSON Key (Admin Access) |
| `GCP_PROJECT_ID` | Google Cloud Project ID |
| `QDRANT_URL` | Qdrant Cluster URL |
| `QDRANT_API_KEY` | Qdrant API Key |
| `QDRANT_COLLECTION_NAME` | Collection Name |
| `EMAIL_HOST` | SMTP Server (e.g., smtp.gmail.com) |
| `EMAIL_PORT` | SMTP Port (e.g., 587) |
| `EMAIL_USERNAME` | Sender Email |
| `EMAIL_PASSWORD` | App Password (Not Login Password) |
| `EMAIL_FROM` | Sender Address |
| `EMAIL_TO` | Notification Recipient |

---

## Verifying Deployment

After a successful pipeline run:

### 1. Backend API (GKE)
Check the status of the Kubernetes Service:
```bash
kubectl get service agent-api-service
```
Use the `EXTERNAL-IP` to test the API:
```bash
curl -X POST "http://<EXTERNAL-IP>/v1/research" \
     -H "Content-Type: application/json" \
     -d '{"query": "Analyze Microsoft revenue", "complexity": "simple"}'
```

### 2. Frontend UI (Cloud Run)
1. Go to **Google Cloud Console > Cloud Run**.
2. Find the `agent-ui` service.
3. Click the **URL** to open the web application.

---

## Project Structure
```
.
├── .github/workflows/     # CI/CD Pipelines
│   └── ci-cd.yaml        # Main Unified Pipeline
├── k8s/                   # Kubernetes Manifests
├── src/
│   ├── agents/           # Agent Logic (Planner, Researcher...)
│   ├── model_validation/ # Validator & Bias Check Scripts
│   ├── tools/            # GCP & Qdrant Clients
│   ├── ui/               # Streamlit App
│   ├── api.py            # FastAPI Backend
│   └── graph.py          # LangGraph Definition
├── Dockerfile            # Backend Image
├── Dockerfile.frontend   # UI Image
└── requirements.txt      # Dependencies
```
