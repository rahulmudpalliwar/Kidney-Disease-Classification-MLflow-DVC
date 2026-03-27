# LangGraph Course - Agentic AI

This repository contains all the code, exercises, and architectural patterns developed for the FreeCodeCamp LangGraph Course. It is designed to take you from basic graph structures to building advanced, stateful multi-agent systems.

---

## Project Overview

**LangGraph** is a Python framework for building stateful, graph-structured AI agent workflows on top of LangChain. This project takes five LangGraph agent patterns and:

1. Wraps them in a **FastAPI** application (`app.py`) as REST endpoints.
2. Packages the application in a **Docker** multi-stage image.
3. Deploys it to a local **Minikube** Kubernetes cluster with a Deployment, NodePort Service, ConfigMap, and Secret.

The Stock Market Performance 2024 PDF (used by the RAG agent) is **bundled inside the Docker image** at `/app/Agents/`, so no external volume mount is required.

---
## Repository Structure

```
agentic_AI/
├── app.py                   # FastAPI app — all 5 agents as REST endpoints
├── Dockerfile               # Multi-stage Docker build (builder + slim runtime)
├── .dockerignore            # Excludes notebooks, caches, secrets from image
├── .env.example             # Template for local .env
├── requirements.txt         # Python dependencies
│
├── Agents/
│   ├── Agent_Bot.py         # Original CLI: stateless chatbot
│   ├── Memory_Agent.py      # Original CLI: chatbot with history
│   ├── ReAct.py             # Original CLI: math tool agent
│   ├── Drafter.py           # Original CLI: document drafting agent
│   ├── RAG_Agent.py         # Original CLI: PDF Q&A (fixed Windows path bug)
│   └── Stock_Market_Performance_2024.pdf   # Bundled PDF for RAG agent
│
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment (1 replica, health probes)
│   ├── service.yaml         # NodePort Service (port 30800)
│   ├── configmap.yaml       # PDF_PATH and CHROMA_PERSIST_DIR config
│   └── secret.yaml          # Secret template (populated by deploy.sh)
│
├── run-all.sh               # One-shot automated deployment script
├── Exercises/               # Jupyter notebooks — LangGraph exercise solutions
└── Graphs/                  # Jupyter notebooks — LangGraph concept demos
```

---

## Core Requirements

| Tool | Install guide |
|---|---|
| Docker | https://docs.docker.com/engine/install/ |
| Minikube | https://minikube.sigs.k8s.io/docs/start/ |
| kubectl | https://kubernetes.io/docs/tasks/tools/ |
| Google Gemini API key | https://aistudio.google.com/apikey (free tier available) |

---

## Deployment

### 1. Clone git repository

```bash
git clone https://github.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC
cd LangGraph_Agentic_AI
```

### 2. Add Google Gemini API key

```bash
echo 'GOOGLE_API_KEY=paste_your_gemini_api_key' > .env
```

> Get a free Gemini API key at https://aistudio.google.com/apikey

### 3.Run the deploy script (does everything automatically)

```bash
bash run-all.sh
```

### 4. Do port-forwarding to expose the service

```bash
kubectl port-forward service/agentic-ai-service 8080:80 &
```

The API is now accessible at **`http://localhost:8080`**.

---

## Accessing the Running API

| URL | Purpose |
|---|---|
| `http://localhost:8080/` | Root — lists all agents |
| `http://localhost:8080/health` | Liveness / readiness probe |
| `http://localhost:8080/docs` | Swagger UI — interactive API explorer |
| `http://localhost:8080/redoc` | ReDoc API documentation |

---

## Verifying & Using Each Agent

### 1. Health Check

```bash
curl http://localhost:8080/health
```

Expected response:
```json
{"status": "healthy", "service": "Agentic AI API"}
```

---

### 2. Agent Bot (Stateless Chatbot)

Single-turn chat. No history is preserved between calls.

```bash
curl -s -X POST http://localhost:8080/api/agent-bot/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "What is LangGraph in one sentence?"}' | python3 -m json.tool
```

Expected response:
```json
{
    "response": "LangGraph is a Python framework ...",
    "session_id": "some-uuid"
}
```

---

### 3. Memory Agent (Multi-turn Chat)

Maintains full conversation history across calls within a session.

**First turn** (no `session_id` — creates a new session):
```bash
curl -s -X POST http://localhost:8080/api/memory/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "My name is Amol"}' | python3 -m json.tool
```

Note the `session_id` in the response, then continue the conversation:

```bash
curl -s -X POST http://localhost:8080/api/memory/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "What is my name?", "session_id": "<session_id_from_above>"}' \
  | python3 -m json.tool
```

View full conversation history:
```bash
curl http://localhost:8080/api/memory/sessions/<session_id> | python3 -m json.tool
```

Clear a session:
```bash
curl -X DELETE http://localhost:8080/api/memory/sessions/<session_id>
```

---

### 4. ReAct Agent (Math Tools)

Uses add / subtract / multiply tools in a ReAct reasoning loop.

```bash
curl -s -X POST http://localhost:8080/api/react/solve \
  -H "Content-Type: application/json" \
  -d '{"query": "Add 40 and 12, then multiply the result by 6"}' \
  | python3 -m json.tool
```

Expected response:
```json
{
    "result": "The result of adding 40 and 12 is 52, and multiplying that by 6 gives 312."
}
```

More examples:
```bash
# Chained operations
-d '{"query": "What is 100 minus 37, then subtract 8 more?"}'

# Mixed operations with explanation
-d '{"query": "Multiply 15 by 4, then add 20 to that result"}'
```

---

### 5. Drafter Agent (Document Drafting)

Creates and iteratively refines a document. The agent uses `update_document` and `save_document` LLM tool calls internally.

**Start a new document:**
```bash
curl -s -X POST http://localhost:8080/api/drafter/chat \
  -H "Content-Type: application/json" \
  -d '{"instruction": "Write a professional introduction about AI in healthcare"}' \
  | python3 -m json.tool
```

Note the `session_id` and `document_content` in the response.

**Add more content to the same document:**
```bash
curl -s -X POST http://localhost:8080/api/drafter/chat \
  -H "Content-Type: application/json" \
  -d '{"instruction": "Add a section on AI diagnostics with 2 bullet points", "session_id": "<session_id>"}' \
  | python3 -m json.tool
```

**Save the document:**
```bash
curl -s -X POST http://localhost:8080/api/drafter/chat \
  -H "Content-Type: application/json" \
  -d '{"instruction": "Save the document as healthcare_ai_report.txt", "session_id": "<session_id>"}' \
  | python3 -m json.tool
```

When saved, the response shows `"is_saved": true`.

---

### 6. RAG Agent (PDF Q&A)

Answers questions about the **Stock Market Performance 2024** PDF bundled in the image. ChromaDB is built on the first query (takes ~20s on first call; subsequent calls are fast).

```bash
curl -s -X POST http://localhost:8080/api/rag/query \
  -H "Content-Type: application/json" \
  -d '{"query": "How did the S&P 500 perform in 2024?"}' \
  | python3 -m json.tool
```

```bash
curl -s -X POST http://localhost:8080/api/rag/query \
  -H "Content-Type: application/json" \
  -d '{"query": "Which sectors had the best performance in 2024?"}' \
  | python3 -m json.tool
```

Expected response format:
```json
{
    "answer": "According to the document (Chunk 1): The S&P 500 ..."
}
```

---

## Swagger UI (Interactive Docs)

Open **`http://localhost:8080/docs`** in your browser for a fully interactive API explorer where you can:
- See all endpoints with request/response schemas
- Send requests directly from the browser
- Inspect example payloads

---

## Operational Commands

```bash
# Check pod status
kubectl get pods -l app=agentic-ai

# Stream live logs
kubectl logs -f deployment/agentic-ai

# Describe pod (debug crash loops)
kubectl describe pod -l app=agentic-ai

# Re-expose after terminal restart
kubectl port-forward service/agentic-ai-service 8080:80 &

# Update API key and restart
export GOOGLE_API_KEY="AIzaSy-..."
kubectl create secret generic agentic-ai-secrets \
    --from-literal=google-api-key="$GOOGLE_API_KEY" \
    --dry-run=client -o yaml | kubectl apply -f -
kubectl rollout restart deployment/agentic-ai
kubectl rollout status deployment/agentic-ai

# Rebuild and redeploy after code changes
eval $(minikube docker-env)
docker build -t agentic-ai:latest .
kubectl rollout restart deployment/agentic-ai

# Scale replicas
kubectl scale deployment agentic-ai --replicas=2

# Tear down all Kubernetes resources
kubectl delete -f k8s/

# Stop Minikube
minikube stop
```

---

### Install all the requirements (for development without Docker):
```bash
# Create a virtual environment (required on Ubuntu/Debian systems)
python3 -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Run the RAG Agent locally (CLI mode)

```bash
# Set up your .env file first
echo 'GOOGLE_API_KEY=AIzaSy-your-key-here' > .env
echo 'PDF_PATH=Agents/Stock_Market_Performance_2024.pdf' >> .env
echo 'CHROMA_PERSIST_DIR=Agents/chroma_db' >> .env

# Run the interactive agent
venv/bin/python Agents/RAG_Agent.py
```

You will see:
```
PDF has been loaded and has 9 pages
Created ChromaDB vector store!

=== RAG AGENT===

What is your question: What stocks performed best in 2024?
Calling Tool: retriever_tool with query: best performing stocks in 2024
Result length: 4831
Tools Execution Complete. Back to the model!

=== ANSWER ===
In 2024, Palantir Technologies (PLTR) was the single best-performing stock...
```

Type `quit` or `exit` to stop the agent.

### Run the full API locally

```bash
venv/bin/python app.py
# API available at http://localhost:8000
```
