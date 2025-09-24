Kubernetes Ingress, Prometheus & Grafana: Command Reference
Below are all the key command lines used in your session, organized by topic. Each command includes a title and a brief description of what it does.

Ingress Setup Commands

1. Install NGINX Ingress Controller (kind setup)

bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/kind/deploy.yaml
Deploys the NGINX ingress controller, recommended for kind clusters.

2. Check Ingress Controller Status

bash
kubectl get pods -n ingress-nginx
Lists all ingress-nginx pods to confirm the controller is running.

3. Expose Ingress Controller via Port-Forwarding

bash
kubectl port-forward svc/ingress-nginx-controller 8080:80 -n ingress-nginx
Forwards port 80 of ingress-nginx-controller to localhost:8080 for testing ingress routes.

4. Apply Your Ingress Routing Rule Resource

bash
kubectl apply -f todo-ingress.yaml
Creates ingress routing rules (YAML) for your backend and frontend services.

Prometheus & Grafana Setup Commands

1. Install Helm (Linux)

bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
Installs Helm v3 on a Linux system.

2. Install Helm (Mac)

bash
brew install helm
Installs Helm using Homebrew on macOS.

3. Add Prometheus Helm Repo

bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
Adds the Prometheus charts repository to your local Helm config.

4. Update Helm Chart Repos

bash
helm repo update
Fetches latest info about available charts from all added repositories.

5. Install Prometheus via Helm

bash
helm install prometheus prometheus-community/prometheus
Deploys Prometheus to your cluster using the community helm chart.

6. Add Grafana Helm Repo

bash
helm repo add grafana https://grafana.github.io/helm-charts
Adds the official Grafana charts repository for Helm.

7. Install Grafana via Helm

bash
helm install grafana grafana/grafana
Deploys Grafana using Helm and its chart.

8. Port-Forward Prometheus

bash
kubectl port-forward svc/prometheus-server 9090:80
Access Prometheus web UI at localhost:9090.

9. Port-Forward Grafana

bash
kubectl port-forward svc/grafana 3000:80
Access Grafana web UI at localhost:3000.

10. Get Grafana Admin Password

bash
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
Retrieves the generated admin password to login to Grafana's web UI.

Helm Utility Commands

1. List All Added Helm Repos

bash
helm repo list
Shows all helm chart repositories added to your client.

2. Search for Charts in a Repo

bash
helm search repo prometheus-community
Lists all available charts from prometheus-community repo.
