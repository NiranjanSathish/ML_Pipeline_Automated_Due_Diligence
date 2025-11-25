# Automated Due Diligence Agent System

An intelligent multi-agent system for financial due diligence and market research, powered by Google Vertex AI (Gemini 2.0 Flash) and Qdrant.

## 🚀 Features

- **Multi-Agent Architecture**:
  - **Analyser Agent**: Decomposes complex queries into focused sub-queries.
  - **Researcher Agent**: Performs hybrid search (Semantic + Keyword) on Qdrant vector database.
  - **Synthesiser Agent**: Generates comprehensive, cited answers.
- **Hybrid Search**: Combines dense embeddings (Vertex AI) with sparse keyword search (BM25) for optimal retrieval.
- **Bias Mitigation**: Automatically detects and mitigates bias against specific groups (e.g., small cap companies) using score boosting.
- **Dockerized**: Fully containerized for easy deployment.
- **Cloud Integration**: Connects to hosted Qdrant Cloud and Google Vertex AI.

## 📋 Prerequisites

- **Docker & Docker Compose**
- **Google Cloud Platform (GCP) Account** with Vertex AI API enabled.
- **Qdrant Cloud Account** (or local instance).
- **Python 3.10+** (for local development).

## 🛠️ Installation

1.  **Clone the repository**:
    ```bash
    git clone <repository-url>
    cd Agents
    ```

2.  **Setup Environment Variables**:
    Copy `.env.example` to `.env` and fill in your details:
    ```bash
    cp .env.example .env
    ```
    *Required variables:*
    - `GCP_PROJECT_ID`
    - `GCP_LOCATION`
    - `QDRANT_URL`
    - `QDRANT_API_KEY`

3.  **GCP Authentication**:
    Place your service account key file as `vertex-key.json` in the project root.

## 🏃 Usage

### Running with Docker (Recommended)

1.  **Build and Start**:
    ```bash
    docker compose up -d --build ml-app
    ```

2.  **Run the Agent Interface**:
    ```bash
    docker compose exec -it ml-app python -m src.main
    ```

3.  **View Logs**:
    - Real-time: `docker logs -f ml-app`
    - File: `logs/system.log`

4.  **View Results**:
    Analysis reports are saved as Markdown files in the `results/` directory.

### Local Development

1.  **Create Virtual Environment**:
    ```bash
    python -m venv venv
    source venv/bin/activate  # Windows: venv\Scripts\activate
    ```

2.  **Install Dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

3.  **Run System**:
    ```bash
    python -m src.main
    ```

## 🏗️ Project Structure

```
├── src/
│   ├── agents/           # Agent implementations (Analyser, Researcher, Synthesiser)
│   ├── model_validation/ # Bias detection and mitigation logic
│   ├── tools/            # Qdrant client, GCP client, Reranker
│   ├── utils/            # Logger
│   ├── config.py         # Central configuration
│   └── main.py           # Entry point
├── logs/                 # Application logs
├── results/              # Generated analysis reports (.md)
├── docker-compose.yml    # Docker orchestration
├── Dockerfile            # Container definition
└── requirements.txt      # Python dependencies
```

## 🛡️ Bias Mitigation

The system includes a `BiasMitigator` that monitors retrieval quality across different groups (e.g., by company ticker). If a group is identified as "under-represented" (score < threshold), the system automatically boosts the retrieval scores for documents belonging to that group to ensure fair coverage.
