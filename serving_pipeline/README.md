

## Serving Cluster & Model Serving

This section guides you through setting up the serving cluster and deploying the model serving stack using local cluster for development and testing.


### 🧩 Deploy KServe & Triton Inference Server

#### 1. Install KServe and Build Triton Image

```bash
kubectx kind-datn-serving
cd serving-cluster
./deploy_kserve.sh
docker build -t tritonserver-datn:v1 . -f Dockerfile.triton
```

#### 2. Deploy Inference Service

```bash
kubectl apply -f inferenceservice-triton-gpu.yaml
```