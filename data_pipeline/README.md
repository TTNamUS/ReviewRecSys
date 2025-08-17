

# Batch Processing
## Installation
### 1. Create Kubernetes Namespaces
- Create namespaces for infrastructure, processor, and airflow.

```shell
kubectl create namespace infrastructure &&
kubectl create namespace processor &&
kubectl create namespace airflow
```

### 2. Install Postgres and MinIO Credentials
- Create Kubernetes secrets for Postgres and MinIO credentials in the specified namespaces.

```shell
kubectl create secret generic postgres-credentials \
  --from-file=config/postgres/postgres-credentials.properties \
  -n infrastructure &&

kubectl create secret generic minio-credentials \
  --from-file=access-key=config/s3/access-key.properties \
  --from-file=secret-key=config/s3/secret-key.properties \
  -n infrastructure &&

kubectl create secret generic minio-credentials \
  --from-file=access-key=config/s3/access-key.properties \
  --from-file=secret-key=config/s3/secret-key.properties \
  -n processor
```

### 3. Install Storage (Postgres and MinIO)
- Deploy Postgres and MinIO using the Helm chart in the `infrastructure` namespace.

```shell
helm upgrade --install storage helm/storage -n infrastructure
```

### 4. Create MinIO Buckets
- Execute Minio pod in the `infrastructure` namespace (**Modify the Minio pod name**).
```shell
kubectl exec -it minio-54776d6f4d-qkdpp -n infrastructure -- bash
```
- Set up the MinIO alias and create buckets for Flink and Kafka tiered storage.
```shell
mc alias set minio http://minio-svc:9000 minio_access_key minio_secret_key
mc mb minio/flink-data &
mc mb minio/kafka-tiered-storage
```














## Set up the batch processing environment.

### 1. Download raw data
- Download raw JSON files from [Amazon Reviews dataset](https://amazon-reviews-2023.github.io/), put **reviews** JSON files to `batch_processing/data/reviews` and **metadata** JSON files for `batch_processing/data/metadata` directory.

### 2. Insert raw JSON files to bronze layer
```shell
kubectl port-forward svc/minio-svc 9000:9000 -n infrastructure
cd batch_processing && python upload_s3.py
```

### 3. Download JAR files
- Download JAR files from [here](https://drive.google.com/file/d/11Wfdw0NSQMK4q8Pkq0auxIM4wNOB6niZ/view?usp=drive_link) and them to `batch_processing/jars` directory.

### 4. Build and push docker image for spark processing
```shell
cd batch_processing
docker build -t <username_dockerhub>/spark_processing:1.1.3 .
docker push <username_dockerhub>/spark_processing:1.1.3
```

### 5. Install Spark Operator
```shell
helm repo add spark-operator https://kubeflow.github.io/spark-operator
helm install spark-operator spark-operator/spark-operator --namespace processor --version 1.2.7 --set serviceAccounts.spark.create=true --set serviceAccounts.spark.name=spark-operator-controller --set sparkJobNamespace=processor --set logLevel=4 --create-namespace
```

## Set up the Airflow orchestration

### 1. Build and push docker image for Airflow pipeline
```shell
docker build -t <username_dockerhub>/airflow_pipeline:1.0.6 k8s/airflow/
docker push <username_dockerhub>/airflow_pipeline:1.0.6
```

### 2. Deploy Airflow orchestration
```shell
helm install airflow apache-airflow/airflow --namespace airflow --create-namespace -f helm/airflow/custom-values.yaml
```

### 3. Grant the permissions for default and airflow-worker service account to access SparkApplication and Job resources
Check permissions:
```shell
kubectl auth can-i get sparkapplications.sparkoperator.k8s.io -n processor --as=system:serviceaccount:airflow:default
kubectl auth can-i get sparkapplications.sparkoperator.k8s.io -n processor --as=system:serviceaccount:airflow:default
kubectl auth can-i create jobs -n processor --as=system:serviceaccount:airflow:airflow-worker
```

### 4. Create configmap for Spark jobs
```shell
kubectl create configmap spark-raw2delta-avro --from-file=k8s/spark/spark_application/raw2delta-avro.yaml -n airflow &&
kubectl create configmap spark-merge2delta --from-file=k8s/spark/spark_application/merge2delta.yaml -n airflow &&
kubectl create configmap spark-adding-uuidv7 --from-file=k8s/spark/spark_application/adding-uuidv7.yaml -n airflow &&
kubectl create configmap spark-generate-silver-schema --from-file=k8s/spark/spark_application/generate-silver-schema.yaml -n airflow &&
kubectl create configmap spark-generate-data-mart --from-file=k8s/spark/spark_application/generate-data-mart.yaml -n airflow &&
kubectl create configmap spark-streaming-data --from-file=k8s/spark/spark_application/streaming-data.yaml -n airflow
```


