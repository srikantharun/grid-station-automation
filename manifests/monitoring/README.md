# Prometheus and Grafana Installation

To install Prometheus and Grafana, use the following Helm command:

```bash
# Add the Prometheus community repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install the kube-prometheus-stack
helm install prometheus prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f manifests/monitoring/prometheus-values.yaml
