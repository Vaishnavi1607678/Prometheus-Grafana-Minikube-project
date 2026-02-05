# Prometheus-Grafana-Minikube-project

This repository provides a **complete guide to deploying a Flask application** on **Minikube**, setting up **Prometheus** for metrics collection, and visualizing data on **Grafana**.  

It’s beginner-friendly and packed with **practical instructions** for a seamless hands-on experience.

##  Table of Contents
1. [Project Overview](#project-overview)
2. [Getting Started with Flask App](#getting-started-with-flask-app)
3. [Deploying Flask App on Minikube](#deploying-flask-app-on-minikube)
4. [Setting Up Prometheus](#setting-up-prometheus)
5. [Configuring Prometheus to Scrape Flask Metrics](#configuring-prometheus-to-scrape-flask-metrics)
6. [Installing Grafana](#installing-grafana)
7. [Visualizing Metrics with Grafana Dashboards](#visualizing-metrics-with-grafana-dashboards)
8. [Highlights](#highlights)



##  Project Overview
This project demonstrates:

- Deploying a Flask application locally and on Minikube.
- Setting up **Prometheus** to scrape metrics from the Flask app.
- Integrating Prometheus with **Grafana** for monitoring and visualization.
- Collecting and visualizing custom metrics such as `total_api_requests_total`.

---

##  Getting Started with Flask App
1. Prepare the Flask app locally.
2. Ensure the app includes **metrics endpoints** for Prometheus to scrape, for example:
```python
from prometheus_client import Counter, generate_latest

REQUEST_COUNT = Counter('total_api_requests_total', 'Total API Requests')

@app.route('/metrics')
def metrics():
    return generate_latest(REQUEST_COUNT)

---
```

##  Deploying Flask App on Minikube

### 1. Build Docker Image

```bash
docker build -t flask-metrics-app:latest .
```

### 2. Start Minikube

```bash
minikube start
```

### 3. Load Docker Image into Minikube

```bash
minikube image load flask-metrics-app:latest
```

### 4. Deploy Flask App

* Create `flask-app.yaml` Kubernetes manifest and apply:

```bash
kubectl apply -f flask-app.yaml
```

### 5. Access the Flask App

```bash
minikube service flask-metrics-app --url
```

---

## 📈 Setting Up Prometheus

### 1. Install Helm

**Windows:**

```bash
winget install Helm.Helm
```

**macOS:**

```bash
brew install helm
```

### 2. Add Prometheus Helm Chart

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 3. Install Prometheus

```bash
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
```

---

## ⚙️ Configuring Prometheus to Scrape Flask Metrics

1. Retrieve Flask App Service IP:

```bash
kubectl get svc
```

2. Edit Prometheus ConfigMap to add your Flask app as a scrape target:

```yaml
scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['<FLASK-APP-IP>:5000']
```

3. Restart Prometheus:

```bash
kubectl rollout restart deployment prometheus-server -n monitoring
```

4. Verify Metrics Collection:

```bash
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
```

* Access Prometheus dashboard at `http://localhost:9090` and search for metrics like `total_api_requests_total`.

---

## 📊 Installing Grafana

### 1. Add Grafana Helm Chart

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 2. Install Grafana

```bash
helm install grafana grafana/grafana -n monitoring --create-namespace
```

### 3. Access Grafana Locally

```bash
kubectl port-forward svc/grafana -n monitoring 3000:80
```

* Default login credentials:

```bash
Username: admin
Password: kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

##  Visualizing Metrics with Grafana Dashboards

1. **Add Prometheus as a Data Source**

   * Navigate to: **Connections → Data Sources → Add Data Source → Prometheus**
   * URL: `http://prometheus-server.monitoring.svc.cluster.local:80`
   * Click **Save & Test**.

2. **Create a Dashboard**

   * Navigate to **Dashboards → New Dashboard → Add New Panel**.
   * Visualize metrics such as `total_api_requests_total`.

---

##  Highlights

* **End-to-End Observability:** Full setup for monitoring Flask app with Prometheus & Grafana.
* **Production-Ready Notes:** Steps for Minikube deployment and scaling considerations.
* **Beginner-Friendly Guide:** Clear step-by-step instructions for first-time users.

---





