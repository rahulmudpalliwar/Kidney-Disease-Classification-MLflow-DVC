# Kidney-Disease-Classification-MLflow-DVC

## Workflows

1. Update config.yaml
2. Update secrets.yaml [Optional]
3. Update params.yaml
4. Update the entity
5. Update the configuration manager in src config
6. Update the components
7. Update the pipeline
8. Update the main.py
9. Update the dvc.yaml
10. Update app.py

### How to run?
### 1) Clone the repository

```bash
https://github.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC
```

### 2) Create a conda environment after opening the repository

```bash
conda create -n kidney python=3.8 -y
```

```bash
conda activate kidney
```

### 3) Install the requirements
```bash
pip install -r requirements.txt
```

### 4) Dagshub
[dagshub](https://dagshub.com/)

MLFLOW_TRACKING_URI=https://dagshub.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC.mlflow \
MLFLOW_TRACKING_USERNAME=rahulmudpalliwar \
MLFLOW_TRACKING_PASSWORD=16a5691ac154a0f6565c636f024a1335a8028faa \
python3 script.py

```bash
export MLFLOW_TRACKING_URI=https://dagshub.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC.mlflow 
export MLFLOW_TRACKING_USERNAME=rahulmudpalliwar 
export MLFLOW_TRACKING_PASSWORD=<api_token> 
```

### 5) MLflow

[Documentation](https://mlflow.org/docs/latest/index.html)

[MLflow tutorial](https://youtu.be/qdcHHrsXA48?si=bD5vDS60akNphkem)

##### cmd
- mlflow ui