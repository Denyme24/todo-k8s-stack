# Kubernetes Observability: Ingress, Prometheus & Grafana

This document provides a reference for setting up observability in Kubernetes using NGINX Ingress, Prometheus, and Grafana.

## Ingress Setup

### 1. Install NGINX Ingress Controller (for kind)

Deploys the NGINX ingress controller, recommended for `kind` clusters.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/kind/deploy.yaml
```

### 2. Check Ingress Controller Status

Lists all `ingress-nginx` pods to confirm the controller is running.

```bash
kubectl get pods -n ingress-nginx
```

### 3. Expose Ingress Controller via Port-Forwarding

Forwards port 80 of the `ingress-nginx-controller` service to `localhost:8080` for local testing.

```bash
kubectl port-forward svc/ingress-nginx-controller 8080:80 -n ingress-nginx
```

### 4. Apply Ingress Routing Rules

Creates ingress routing rules for your backend and frontend services from a YAML file.

```bash
kubectl apply -f todo-ingress.yaml
```

---

## Prometheus & Grafana Setup

### 1. Install Helm

**On Linux:**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**On macOS:**

```bash
brew install helm
```

### 2. Add Helm Repositories

**Prometheus:**

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

**Grafana:**

```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

### 3. Update Helm Repositories

Fetches the latest chart information from all added repositories.

```bash
helm repo update
```

### 4. Install Prometheus & Grafana

**Prometheus:**

```bash
helm install prometheus prometheus-community/prometheus
```

**Grafana:**

```bash
helm install grafana grafana/grafana
```

---

## Accessing Services

### 1. Port-Forward Prometheus

Access the Prometheus UI at `http://localhost:9090`.

```bash
kubectl port-forward svc/prometheus-server 9090:80
```

### 2. Port-Forward Grafana

Access the Grafana UI at `http://localhost:3000`.

```bash
kubectl port-forward svc/grafana 3000:80
```

### 3. Get Grafana Admin Password

Retrieves the auto-generated admin password for Grafana.

```bash
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

---

## Helm Utility Commands

### 1. List Helm Repositories

Shows all Helm chart repositories added to your client.

```bash
helm repo list
```

### 2. Search for Charts

Searches for available charts in a specific repository (e.g., `prometheus-community`).

```bash
helm search repo prometheus-community
```
