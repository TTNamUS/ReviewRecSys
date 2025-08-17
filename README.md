# ReviewRecSys: A Recommendation System with MLOps on Kubernetes

A scalable recommendation system built from the ground up using the Amazon Reviews dataset. This project implements a complete MLOps pipeline, from data ingestion and feature engineering to distributed training, serving, and monitoring. It provides both a RESTful API and a user-friendly interface for real-time interaction.

## ✨ Key Features

*   **End-to-End MLOps Pipeline:** A fully automated workflow for the entire machine learning lifecycle, covering data ingestion, feature engineering, distributed training, model serving, and monitoring.
*   **Two-Stage Recommendation Model:**
    *   **Candidate Generation:** Utilizes `Item2Vec` to learn dense item embeddings for efficient retrieval of relevant candidates.
    *   **Personalized Ranking:** Employs a sequence-aware model that leverages temporal user behavior and engineered features for fine-tuned, personalized recommendations.
*   **Scalable:** Built on Kubernetes to ensure scalability, portability, and resilience.
*   **Interactive API & UI:** Exposes recommendation services through a clean API and includes a simple web UI for demonstration and user interaction.

## ⚙️ Technology Stack

This project leverages a modern stack of tools for building robust MLOps pipelines:

*   **Workflow Orchestration:** **Airflow**
*   **Data Processing & Feature Engineering:** **Spark**
*   **Storage:**
    *   **MinIO:** S3-compatible object storage for datasets and artifacts.
    *   **Postgres:** Relational database for metadata and structured data.
    *   **Redis:** In-memory data store for caching and session management.
    *   **Qdrant:** Vector database for efficient similarity search on item embeddings.
*   **ML & Distributed Training:**
    *   **Ray:** For distributed computing and training workloads.
    *   **Kubeflow:** For orchestrating ML workflows on Kubernetes.
    *   **MLflow:** For experiment tracking, model registry, and lifecycle management.
*   **Model Serving:** **KServe / Triton Inference Server**
*   **Infrastructure:** **Kubernetes**

## 🏗️ Architecture Overview

The system is designed as a series of interconnected components forming a complete MLOps pipeline:

1.  **Data Ingestion:** Raw data from the Amazon Reviews dataset is ingested and stored in MinIO.
2.  **Feature Engineering:** Spark jobs process the raw data, creating user and item features that are stored in Postgres and Redis. Item embeddings are generated and loaded into Qdrant.
3.  **Training Pipeline:** A two-stage training process orchestrated by Kubeflow and managed by MLflow.
    *   **Stage 1 (Candidate Generation):** An `Item2Vec` model is trained on user-item interaction sequences to generate item embeddings.
    *   **Stage 2 (Ranking):** A sequence-aware model is trained using Ray for distributed processing. This model uses user interaction history and other features to rank the candidates generated in the first stage.
4.  **Model Serving:** The trained models are deployed using KServe and Triton Inference Server on Kubernetes, providing a scalable and efficient inference endpoint.
5.  **Monitoring:** LGTM Stack (Loki – log aggregation, Grafana – visualization & alerting, Tempo – tracing, Mimir – metrics storage & querying)
6.  **API & UI:** A FastAPI application provides an API gateway, and a simple Gradio UI allows users to interact with the recommendation system.

## 📚 Dataset

> Amazon Reviews 2023 – Toys and Games (Metadata + Reviews)

This project uses a combination of product metadata and user reviews for the "Toys and Games" category from the [Amazon Reviews 2023](https://huggingface.co/datasets/McAuley-Lab/Amazon-Reviews-2023) dataset by the McAuley Lab.

## ⚙️ Pipelines

The core of this project is a set of automated pipelines for data processing, model training, and serving.

### Data Pipeline

The data pipeline is responsible for ingesting raw data, processing it with Spark, and organizing it into a multi-layered data lake (Bronze, Silver, Gold) on MinIO. The entire workflow is orchestrated by Airflow.

For detailed instructions on setting up the infrastructure (Kubernetes, Postgres, MinIO) and running the data processing jobs, please refer to the **[Data Pipeline README](data_pipeline/README.md)**.

### Training Pipeline

The training pipeline is orchestrated using **Kubeflow** to automate the entire model development lifecycle. It leverages a **Ray Cluster** on Kubernetes for distributed, high-performance training of the sequence-aware ranking model.

**MLflow** is integrated for robust experiment tracking, artifact storage, and model registration. A **Jenkins** CI/CD pipeline automates the build and deployment process, triggered by model promotions in MLflow. The pipeline also handles offline caching by loading item embeddings into **Redis** from a **Qdrant** vector database to ensure low-latency inference.

For detailed instructions on setting up the environment and running the training workflows, please refer to the **[Training Pipeline README](training_pipeline/README.md)**.

### Serving Pipeline

The serving pipeline leverages **KServe** and **Triton Inference Server** to deploy the trained recommendation models on Kubernetes. This setup provides a scalable, high-performance, and standardized inference endpoint for both the candidate generation and ranking models.

For detailed deployment instructions, please refer to the **[Serving Pipeline README](serving_pipeline/README.md)**.

## 📊 Monitoring

The system is monitored using the **LGTM Stack** (Loki, Grafana, Tempo, Mimir), providing comprehensive observability. **Grafana** dashboards are used to visualize key system metrics and model performance, while **Loki** aggregates logs from all components. Performance and load testing are conducted with **Locust** to ensure the system's reliability and scalability under pressure.

For setup instructions, please refer to the **[Monitoring README](monitoring/README.md)**.

## 🌐 API Gateway & UI

### API Gateway

The API Gateway provides a single entry point for all client requests.

```bash
cd api_gateway
docker build -t <username_dockerhub>/api-gateway:v1 .
kind load docker-image <username_dockerhub>/api-gateway:v1 --name datn-serving
kubectl create ns api-gateway
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl port-forward svc/api-gateway-service 8009:80 -n api-gateway
```

### UI

You can run the UI locally to interact with the system. First, create a virtual environment and install the dependencies:
```bash
pip install -r requirements.txt
```

Start the app:
```bash
python main.py
```

Access the UI by opening [http://localhost:7860](http://localhost:7860) in your browser.

![Demo UI](img/demo_ui.gif)
