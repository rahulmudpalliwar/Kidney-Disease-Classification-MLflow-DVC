# Kidney-Disease-Classification-MLflow-DVC

A high-performance MLOps ecosystem designed to classify Kidney CT scans into Normal or Tumor categories with clinical-grade precision.

By leveraging VGG16 Transfer Learning, this project bridges the gap between deep learning research and production-ready infrastructure. It features a fully automated, local-first deployment stack—eliminating cloud costs while maintaining enterprise standards for reproducibility and scalability.

## Architecture & Workflow
ML Pipeline (DVC DAG)
The project is structured into four reproducible stages managed by DVC:

1. **Data Ingestion**: Downloads the CT kidney dataset from Google Drive to artifacts/data_ingestion/.

2. **Prepare Base Model**: Loads VGG16 (ImageNet weights) and attaches a custom classification head.

3. **Model Training**: Fine-tunes the model using data augmentation (rotation, zoom, flips).

4. **Evaluation + MLflow**: Calculates final metrics and logs parameters, metrics, and the model artifact to a local MLflow tracking server.

```
+----------------+            +--------------------+ 
| data_ingestion |            | prepare_base_model | 
+----------------+*****       +--------------------+ 
         *             *****             *           
         *                  ******       *           
         *                        ***    *           
         **                        +----------+      
           **                      | training |      
             ***                   +----------+      
                ***             ***                  
                   **         **                     
                     **     **                       
                  +------------+                     
                  | evaluation |                     
                  +------------+  
```

## Project Structure
```
.
├── README.md
├── app.py
├── config
│   └── config.yaml
├── dvc.lock
├── dvc.yaml
├── inputImage.jpg
├── logs
│   └── running_logs.log
├── main.py
├── model
│   ├── model.h5
│   └── model.h5:Zone.Identifier
├── params.yaml
├── requirements.txt
├── research
│   ├── 01_data_ingestion.ipynb
│   ├── 02_prepare_base_model.ipynb
│   ├── 03_model_training.ipynb
│   ├── 04_model_evaluation_with_mlflow.ipynb
│   └── trials.ipynb
├── scores.json
├── setup.py
├── src
│   ├── cnnClassifier
│   │   ├── __init__.py
│   │   ├── __pycache__
│   │   │   └── __init__.cpython-38.pyc
│   │   ├── components
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-38.pyc
│   │   │   │   ├── data_ingestion.cpython-38.pyc
│   │   │   │   ├── model_evaluation_mlflow.cpython-38.pyc
│   │   │   │   ├── model_training.cpython-38.pyc
│   │   │   │   └── prepare_base_model.cpython-38.pyc
│   │   │   ├── data_ingestion.py
│   │   │   ├── model_evaluation_mlflow.py
│   │   │   ├── model_training.py
│   │   │   └── prepare_base_model.py
│   │   ├── config
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-38.pyc
│   │   │   │   └── configuration.cpython-38.pyc
│   │   │   └── configuration.py
│   │   ├── constants
│   │   │   ├── __init__.py
│   │   │   └── __pycache__
│   │   │       └── __init__.cpython-38.pyc
│   │   ├── entity
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-38.pyc
│   │   │   │   └── config_entity.cpython-38.pyc
│   │   │   └── config_entity.py
│   │   ├── pipeline
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── __init__.cpython-38.pyc
│   │   │   │   ├── prediction.cpython-38.pyc
│   │   │   │   ├── stage_01_data_ingestion.cpython-38.pyc
│   │   │   │   ├── stage_02_prepare_base_model.cpython-38.pyc
│   │   │   │   ├── stage_03_model_training.cpython-38.pyc
│   │   │   │   └── stage_04_model_evaluation.cpython-38.pyc
│   │   │   ├── prediction.py
│   │   │   ├── stage_01_data_ingestion.py
│   │   │   ├── stage_02_prepare_base_model.py
│   │   │   ├── stage_03_model_training.py
│   │   │   └── stage_04_model_evaluation.py
│   │   └── utils
│   │       ├── __init__.py
│   │       ├── __pycache__
│   │       │   ├── __init__.cpython-38.pyc
│   │       │   └── common.cpython-38.pyc
│   │       └── common.py
│   └── cnnClassifier.egg-info
│       ├── PKG-INFO
│       ├── SOURCES.txt
│       ├── dependency_links.txt
│       └── top_level.txt
├── template.py
└── templates
    └── index.html
```

## Tech Stack

| Layer | Technology |
| :--- | :--- |
| **Deep Learning** | TensorFlow / Keras (VGG16) |
| **Experiment Tracking** | MLflow |
| **Data Versioning** | DVC (Data Version Control) |
| **Web Framework** | Flask |
| **Language** | Python 3.8+ |
| **Environment** | Conda / Miniconda |


## Installation & Setup

1. Clone the Repository
```bash
git clone https://github.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC
cd Kidney-Disease-Classification
```

2. Create Virtual Environment
```Bash
# Using Conda
conda create -n kidney python=3.8 -y
conda activate kidney

# Or using venv
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```
3. Install Dependencies
```Bash
pip install -r requirements.txt
pip install -e .
```
4. Dagshub
Get you credentials from: [dagshub](https://dagshub.com/) & run below in terminal

```bash
export MLFLOW_TRACKING_URI=https://dagshub.com/rahulmudpalliwar/Kidney-Disease-Classification-MLflow-DVC.mlflow 
export MLFLOW_TRACKING_USERNAME=rahulmudpalliwar 
export MLFLOW_TRACKING_PASSWORD=<api_token> 
```
5. Run all 4 modules using single file
```bash
python3 main.py
```
6. Run the Flask app
```bash
python3 app.py
```
we can avigate to http://localhost:8080

## Run Training Pipeline

- Via DVC (locally)
```bash
dvc repro          # runs all 4 stages (skips up-to-date stages)
dvc dag            # visualise the pipeline DAG
```
- Via Web UI
Navigate to http://localhost:8080/train

- Via main.py
```bash
python3 main.py
```
Pipeline runs all 4 stages in below order:

- Data Ingestion — downloads data.zip from Google Drive, extracts to artifacts/data_ingestion/kidney-ct-scan-image/
- Prepare Base Model — downloads VGG16 weights (ImageNet), adds Dense(2, softmax) head, saves to artifacts/prepare_base_model/
- Model Training — trains for 10 epochs with augmentation, saves to artifacts/training/model.h5
- Model Evaluation — evaluates on validation set, logs loss + accuracy to MLflow, registers model as VGG16Model

## Make Predictions 
1. Run command "dvc repro"
It will pop up to open browser UI
OR 
Open http://localhost:8080

2. Click Upload → select a kidney CT scan image (JPEG/PNG) (Normal/Tumor)
3. Click Predict → result appears in the Prediction Results panel

## Output
Real CT scans from the training dataset includes with below results:
| Normal kidney | `{"image": "Normal"}` |
| Tumor kidney | `{"image": "Tumor"}` |


## MLflow

[Documentation](https://mlflow.org/docs/latest/index.html)

[MLflow video tutorial- YouTube](https://youtu.be/qdcHHrsXA48?si=bD5vDS60akNphkem)

## DVC 
```bash
dvc init           # initialise DVC (already done)
dvc repro          # reproduce pipeline (only runs changed stages)
dvc dag            # print the pipeline DAG
```

## Developer Guide Workflows (Setting up a New Pipeline Stage)

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

############################# *THE END* #############################
