# NYC Taxi Tip Prediction with Ray & XGBoost Ensemble

This project demonstrates how to scale an end-to-end Machine Learning pipeline—from hyperparameter tuning and distributed training to ensemble serving and batch inference—using Ray and XGBoost on the New York City (NYC) Taxi dataset.

---

## 🎯 Project Goal

The primary goal of this project is to train and deploy an ensemble XGBoost model using the NYC Taxi dataset to predict trip tips (`tip_amount`). 

Starting from a vanilla, single-node XGBoost baseline, the codebase scales the pipeline to:
1. **Tune**: Find optimal hyperparameters in parallel.
2. **Train**: Perform distributed training on multiple workers.
3. **Serve**: Deploy an ensemble model serving pipeline accessible via HTTP.
4. **Data**: Execute parallel batch inference on large datasets.

---

## 🛠️ Tech Stack

*   **Ray Core & Ray AI Libraries**:
    *   **Ray Tune**: Distributed hyperparameter tuning.
    *   **Ray Train**: Distributed model training across multiple workers.
    *   **Ray Serve**: Multi-deployment model serving with ensemble router pattern.
    *   **Ray Data**: High-throughput distributed data loading and batch inference.
*   **FastAPI & Pydantic**: Web API ingress layer and type-safe payload validation for serving.
*   **XGBoost**: Gradient boosted decision trees model framework.
*   **Conda / Miniforge**: Environment management, optimized for macOS ARM64/Apple Silicon.
*   **UV**: Extremely fast Python package installer and resolver.

---

## 📊 Dataset Setup

To keep this repository lightweight and reproducible, the raw data files are not included in the repository. 

This project utilizes the June 2021, January 2021, and February 2021 files from the **New York City TLC Trip Record Data**. Before running the notebook, you must download these files and place them into your local project directory.

1. **Download the required `.parquet` files:**
   - [June 2021 Taxi Data (Main Notebook)](https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-06.parquet)
   - [January 2021 Taxi Data (Ray Train Worker 0)](https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet)
   - [February 2021 Taxi Data (Ray Train Worker 1)](https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-02.parquet)

2. **Organize your local directory:**
   Place the downloaded files directly into the root folder where your Jupyter Notebook lives:
   ```text
   ray-xgboost-e2e-pipeline/
   ├── Intro_Ray_AI_Libs_Overview_Fresh.ipynb
   ├── yellow_tripdata_2021-01.parquet
   ├── yellow_tripdata_2021-02.parquet
   └── yellow_tripdata_2021-06.parquet
   ```

## 🚀 How to Run It

Follow these step-by-step instructions to set up your environment and run the notebook:

### 1. Clone the repository:
   ```bash
   git clone [https://github.com/alwin-eldhose/ray-xgboost-e2e-pipeline.git](https://github.com/alwin-eldhose/ray-xgboost-e2e-pipeline.git)
   cd ray-xgboost-e2e-pipeline
   ```

### 2. Prerequisites (Install Conda/Miniforge)
If you do not have Conda installed, we recommend **Miniforge** (especially for macOS Apple Silicon / ARM64):

```bash
# Download and install Miniforge
curl -LO https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
bash Miniforge3-MacOSX-arm64.sh
```
Follow the prompts to complete the installation and restart your terminal.

### 3. Create the Conda Environment
Create a new conda environment named `ray-jupyter` with Python 3.11:

```bash
conda create -n ray-jupyter python=3.11 -y
```

### 4. Activate the Environment
```bash
conda activate ray-jupyter
```

### 5. Install dependencies using `uv`
For reproducibility, we use the `requirements.lock` file to install dependencies via `uv`:

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install locked dependencies
uv pip install -r requirements.lock
```

*Note: On macOS, if you encounter an `XGBoostError` during imports, you may need to run `brew install libomp` in your terminal.*

### 6. Register the Conda Environment in Jupyter (Optional but Recommended)
To select your custom environment as the Jupyter kernel:

```bash
pip install ipykernel
python -m ipykernel install --user --name ray-jupyter --display-name "Python (ray-jupyter)"
```

### 7. Launch Jupyter Notebook
```bash
jupyter notebook
```

### 8. Execute the Pipeline Notebook
Open **`Intro_Ray_AI_Libs_Overview_Fresh.ipynb`** in Jupyter Notebook or VS Code, select the `Python (ray-jupyter)` kernel, and run the cells sequentially. The notebook executes:
- Data ingestion and preparation.
- Baseline XGBoost training.
- Hyperparameter tuning using Ray Tune.
- Distributed training using Ray Train.
- Deploying the XGBoost model using Ray Serve.
- Scalable batch offline prediction with Ray Data.

---

## ⚡ Querying the Live API

Once the Ray Serve deployment cell is run, the ensemble model is served locally at `http://localhost:8000/ensemble/predict`. 

Here is how an engineer would query the live API using standard tools:

### Option A: Using `curl`
```bash
curl -X POST http://localhost:8000/ensemble/predict \
     -H "Content-Type: application/json" \
     -d '{
       "passenger_count": 1,
       "trip_distance": 2.5,
       "fare_amount": 10.0,
       "tolls_amount": 0.5
     }'
```

### Option B: Using Python `requests`
```python
import requests

url = "http://localhost:8000/ensemble/predict"
payload = {
    "passenger_count": 1,
    "trip_distance": 2.5,
    "fare_amount": 10.0,
    "tolls_amount": 0.5
}

response = requests.post(url, json=payload)
result = response.json()

print("Status Code:", response.status_code)
print("Prediction Response:", result)
# Example Output: {'prediction': 2.0076115131378174}
```
