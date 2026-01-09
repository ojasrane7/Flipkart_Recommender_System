# Flipkart Recommender System (RAG + Flask)

A Flask-based RAG (Retrieval-Augmented Generation) application that uses:
- **Groq** for LLM inference
- **Hugging Face** for embeddings
- **AstraDB Vector Store** for retrieval
- **Docker + Minikube (Kubernetes)** for deployment
- **Prometheus + Grafana** for monitoring

---

## Repository Structure (high level)

- `app.py` — Flask entrypoint
- `flipkart/` — ingestion + RAG chain code
- `templates/` — HTML templates (UI)
- `static/` — static assets
- `Dockerfile` — container build
- `flask-deployment.yaml` — Kubernetes deployment + service for the app
- `prometheus/` — Prometheus config + deployment YAMLs
- `grafana/` — Grafana deployment YAMLs
- `requirements.txt` — Python dependencies

---

## Prerequisites

### Accounts / Tokens
You will need:
- `GROQ_API_KEY`
- `ASTRA_DB_APPLICATION_TOKEN`
- `ASTRA_DB_API_ENDPOINT`
- `ASTRA_DB_KEYSPACE` (default: `default_keyspace`)
- `HF_TOKEN`
- `HUGGINGFACEHUB_API_TOKEN`

> ⚠️ Never commit `.env` files or tokens. Use Kubernetes secrets.

### Local / VM Tools
- Git
- Docker
- Minikube
- kubectl

---

## 1) Create a Google Cloud VM (Ubuntu 24.04)

Recommended VM settings:
- **Series:** E2  
- **Machine type:** Standard (16 GB RAM)
- **Disk:** 256 GB
- **OS:** Ubuntu 24.04 LTS
- Enable **HTTP** and **HTTPS** traffic  
Connect via the browser **SSH** button.

---

## 2) Clone the Repo on the VM

```bash
git clone https://github.com/ojasrane7/Flipkart_Recommender_System.git
cd Flipkart_Recommender_System
ls
```

---

## 3) Install Docker (Ubuntu)

Install using the official Docker docs for Ubuntu, then verify:

```bash
docker run hello-world
```

Enable Docker to start on boot:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Check Docker status:

```bash
systemctl status docker
docker ps
docker ps -a
```

---

## 4) Install Minikube + kubectl

### Minikube
Install from official Minikube docs (Linux x86_64). Then:

```bash
minikube start
```

### kubectl (snap)
```bash
sudo snap install kubectl --classic
kubectl version --client
```

Validate cluster:

```bash
minikube status
kubectl get nodes
kubectl cluster-info
docker ps
```

---

## 5) Build & Deploy the App (Kubernetes on Minikube)

### Point Docker to Minikube
```bash
eval $(minikube docker-env)
```

### Build the image
```bash
docker build -t flask-app:latest .
```

### Create Kubernetes Secrets
Replace placeholders with your real credentials:

```bash
kubectl create secret generic llmops-secrets   --from-literal=GROQ_API_KEY="..."   --from-literal=ASTRA_DB_APPLICATION_TOKEN="..."   --from-literal=ASTRA_DB_KEYSPACE="default_keyspace"   --from-literal=ASTRA_DB_API_ENDPOINT="..."   --from-literal=HF_TOKEN="..."   --from-literal=HUGGINGFACEHUB_API_TOKEN="..."
```

### Deploy
```bash
kubectl apply -f flask-deployment.yaml
kubectl get pods
kubectl get svc
```

### Access the App
```bash
kubectl port-forward svc/flask-service 5000:80 --address 0.0.0.0
```

Open in browser:
- `http://<VM_EXTERNAL_IP>:5000`

---

## 6) Monitoring: Prometheus + Grafana

### Create namespace
```bash
kubectl create namespace monitoring
kubectl get ns
```

### Deploy Prometheus + Grafana
```bash
kubectl apply -f prometheus/prometheus-configmap.yaml
kubectl apply -f prometheus/prometheus-deployment.yaml
kubectl apply -f grafana/grafana-deployment.yaml
```

### Prometheus UI
```bash
kubectl port-forward --address 0.0.0.0 svc/prometheus-service -n monitoring 9090:9090
```

Open:
- `http://<VM_EXTERNAL_IP>:9090`

### Grafana UI
```bash
kubectl port-forward --address 0.0.0.0 svc/grafana-service -n monitoring 3000:3000
```

Open:
- `http://<VM_EXTERNAL_IP>:3000`

Default login (if unchanged in your manifest):
- **admin / admin**

### Add Prometheus datasource in Grafana
1. Grafana → **Settings** → **Data Sources** → **Add Data Source**
2. Choose **Prometheus**
3. URL:
   - `http://prometheus-service.monitoring.svc.cluster.local:9090`
4. **Save & Test**

---

## Troubleshooting

### Pod not running
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Service issues
```bash
kubectl get svc
kubectl describe svc flask-service
```

### Restart Minikube
```bash
minikube delete
minikube start
```

---

## Security Notes
- Never push `.env` / `.env.bak` / tokens to GitHub
- Rotate keys if they were ever exposed
- Use Kubernetes secrets for credentials
