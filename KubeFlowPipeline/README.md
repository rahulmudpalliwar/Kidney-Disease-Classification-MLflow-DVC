# KubeFlow Pipelines (KFP) — Local MLOps Training

A comprehensive guide to deploying and orchestrating machine learning workflows locally using Kubeflow Pipelines v2, Kind (Kubernetes in Docker), and Argo Workflows. This project focuses on building a reproducible, containerized environment for MLOps without cloud overhead.

## System Architecture

| Component	| Technology | Role |
|---|---|---|
| Orchestrator |	Kubeflow Pipelines (KFP) v2 |	Workflow management & UI |
| Engine |	Argo Workflows |	CRD-based task execution |
| Local Cluster |	Kind (Kubernetes in Docker) |	Lightweight K8s environment |
| Storage |	MinIO / SeaweedFS |	Artifact & Metadata storage |
| Database |	MySQL |	Metadata & Experiment tracking |


## Project Structure

```
kubeFlowPipeline/
├── hello_pipeline.py          # Hello World pipeline definition (KFP component + pipeline)
├── hello_world_pipeline.yaml  # Compiled IR YAML for hello_pipeline
├── iris_pipeline.py           # Iris ML pipeline (load data → train model)
├── iris_pipeline.yaml         # Compiled IR YAML for iris_pipeline
└── README.md                  # This file
```

---

## Prerequisites

### Local Machine

| Requirement | Version | Notes |
|---|---|---|
| Python | >= 3.8 | 3.10 recommended |
| pip | latest | `pip install --upgrade pip` |
| kfp SDK | 2.16.0 | Kubeflow Pipelines SDK v2 |
| kfp-kubernetes | latest | For security context support |

---

## Deployment Guide

### 1. Clone / Navigate to the Project

```bash
git clone https://github.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC.git
cd Kidney-Disease-Classification-MLflow-DVC/KubeFlowPipeline
```

### 2. Create a Virtual Environment (Recommended)

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Python Dependencies

```bash
pip install --upgrade pip
pip install kfp==2.16.0
pip install kfp-kubernetes
```

> **Note:** `kfp-kubernetes` is required for `kubernetes.set_security_context()` used in both pipelines.

### 4. Verify Installation

```bash
python -c "import kfp; print(kfp.__version__)"
# Expected: 2.16.0

python -c "from kfp import kubernetes; print('kfp-kubernetes OK')"
```

### 5. Initialize the Cluster

```bash
kind create cluster --name=kfp-demo-yt
```
### 5. Install KFP Manifests

```bash
export PIPELINE_VERSION=2.15.0
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/dev?ref=$PIPELINE_VERSION"
```

---

## Developing & Compiling Pipelines

### 1. Hello World Pipeline
- create hello_pipeline.py 
- run below to create hello_world_pipeline.yaml
```bash
python3 hello_pipeline.py
```
### 2. Iris Classification Pipeline
- create iris_pipeline.py
- run below to create iris_pipeline.yaml
```bash
python3 iris_pipeline.py
```
---
## Accessing the UI
Port-forward the UI service to access the dashboard at localhost:8080:
```bash
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80
```

---

## Workflow

```
1. Write Pipeline
      │
      │  Define @dsl.component functions
      │  Define @dsl.pipeline orchestration
      │  Apply kubernetes.set_security_context()
      │
      ▼
2. Compile Pipeline
      │
      │  compiler.Compiler().compile(pipeline_func, 'output.yaml')
      │
      ▼
3. Upload to Kubeflow UI / CLI
      │
      │  Open KFP UI → Pipelines → Upload Pipeline → select .yaml
      │  OR use kfp.Client() to upload programmatically
      │
      ▼
4. Create a Run
      │
      │  Select pipeline version → Create Run
      │  Provide input parameters (e.g., recipient="Alice")
      │
      ▼
5. Monitor Execution
      │
      │  View DAG graph in KFP UI
      │  Click each node to see logs, inputs, outputs
      │
      ▼
6. Inspect Results
         View output parameters / metrics in the Run details panel
```

## Running Pipelines on Kubeflow (Kubeflow UI Manual)

1. Open the Kubeflow Pipelines dashboard (e.g., `http://<kubeflow-host>/pipeline`)
2. Go to **Pipelines** → **Upload Pipeline**
3. Upload `hello_world_pipeline.yaml` or `iris_pipeline.yaml`
4. Go to **Runs** → **Create Run**
5. Select the uploaded pipeline, set parameters, and submit

---

## Common Commands & Debugging

| Goal |	Command |
|---|---|
| Check Pods| 	kubectl get pods -n kubeflow |
| Describe Failure |	kubectl describe pod <pod-name> -n kubeflow |
| Delete All Runs |	kubectl delete workflows --all -n kubeflow |
| View Logs |	kubectl logs -n kubeflow deploy/ml-pipeline |

---


############################## *THE END* ##############################