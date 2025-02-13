# lgtm-stack-k3s

My LGTM (Loki, Grafana, Tempo and Mimir) stack along with OpenTelemetry for testing observability signals locally.

## Prerequisites

- k3s
- Git
- Helm

```shell
# Install k3s node
curl -sfL https://get.k3s.io | sh -

# Check node is working
kubectl get pods

# Install Git
sudo apt install git-all

# Install Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Install

```shell


# Add OTel chart repository
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

# Add LGTM charts repositories
helm repo add grafana https://grafana.github.io/helm-charts

# Build dependencies
helm dependency build

# Install chart
helm --kubeconfig /etc/rancher/k3s/k3s.yaml install lgtm-stack .

# Uninstall chart
helm --kubeconfig /etc/rancher/k3s/k3s.yaml uninstall lgtm-stack
```