# observability-stack-k3s

My observability stack for testing observability signals locally.

This contains:
- OpenTelemetry Collector, for flowing observability signals
- Grafana, to visualize observability signals
- Loki, to explore logs
- Tempo, to explore traces
- Prometheus, to explore metrics
- Pyroscope, to explore profiles

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

# Add Grafana charts repositories
helm repo add grafana https://grafana.github.io/helm-charts

# Add Prometheus chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Build dependencies
helm dependency build

# Install chart
helm --kubeconfig /etc/rancher/k3s/k3s.yaml upgrade --install observability-stack .

# Uninstall chart
helm --kubeconfig /etc/rancher/k3s/k3s.yaml uninstall observability-stack
```

## Test

```shell
# Get Grafana password
kubectl get secret --namespace default observability-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# Send test trace
curl -X POST http://otel.local/v1/traces \
     -H "Content-Type: application/json" \
     -d '{
  "resourceSpans": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.name",
            "value": { "stringValue": "test-service" }
          }
        ]
      },
      "scopeSpans": [
        {
          "scope": {
            "name": "test-tracer",
            "version": "1.0"
          },
          "spans": [
            {
              "traceId": "'$(uuidgen | tr -d -)'",
              "spanId": "'$(uuidgen | tr -d - | cut -c1-16)'",
              "name": "test-trace",
              "kind": 1,
              "startTimeUnixNano": "'$(date +%s%N)'",
              "endTimeUnixNano": "'$(($(date +%s%N) + 1000000))'",
              "attributes": [
                {
                  "key": "test.timestamp",
                  "value": { "stringValue": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'" }
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}'

# Send test metric
curl -X POST http://otel.local/v1/metrics \
     -H "Content-Type: application/json" \
     -d '{
  "resourceMetrics": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.name",
            "value": { "stringValue": "test-service" }
          }
        ]
      },
      "scopeMetrics": [
        {
          "scope": {
            "name": "test-metrics",
            "version": "1.0"
          },
          "metrics": [
            {
              "name": "test_metric",
              "description": "A test metric sent via curl",
              "unit": "1",
              "sum": {
                "dataPoints": [
                  {
                    "timeUnixNano": "'$(date +%s%N)'",
                    "asDouble": 42.0,
                    "attributes": [
                      {
                        "key": "test.timestamp",
                        "value": { "stringValue": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'" }
                      }
                    ]
                  }
                ],
                "aggregationTemporality": 2,
                "isMonotonic": false
              }
            }
          ]
        }
      ]
    }
  ]
}'

# Send test log
curl -X POST http://otel.local/v1/logs \
-H "Content-Type: application/json" \
-d '{
  "resourceLogs": [
    {
      "resource": {
        "attributes": [
          { "key": "service.name", "value": { "stringValue": "curl-service" } },
          { "key": "host.name", "value": { "stringValue": "localhost" } }
        ]
      },
      "scopeLogs": [
        {
          "scope": {
            "name": "curl-log-scope"
          },
          "logRecords": [
            {
              "timeUnixNano": "'$(date +%s%N)'",
              "severityNumber": 9,
              "severityText": "INFO",
              "body": { "stringValue": "This is a test log from curl to OpenTelemetry Collector" },
              "attributes": [
                { "key": "http.method", "value": { "stringValue": "GET" } },
                { "key": "http.url", "value": { "stringValue": "https://example.com" } }
              ]
            }
          ]
        }
      ]
    }
  ]
}'
```
