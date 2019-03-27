# Kubow - An Architecture-Based Self-Adaptation Service for Kubernetes Applications

The main goal of Kubow is to provide an architecture-based self-adaptation solution for containerized Kubernetes applications. The proposed solution is a Docker and Kubernetes customization of the Rainbow architecture-based self-adaptation framework, originally developed at Carnegie Mellon University. 

Kubow has been preliminarily evaluated both qualitatively and quantitatively. The qualitative results show that the proposed solution cab be easy to use and customize for different applications, and ease to extend to support different types of adaptation and different architectural models. The quantitative results, in turn, obtained in the Amazon cloud, show that the proposed solution is effective in dynamically adapting the behavior of a Kubernetes application, in response to change in its demand. 

## Getting started

### Pre-requisites

- Install kubectl
- A running Kubernetes Cluster

### Setting up namespaces
```sh
kubectl apply -f ./namespaces.yaml
```

### Setting up kube-znn 
```sh
kubectl apply -f ./kube-znn
```


### Setting up the Monitoring Stack

Starting metrics server

```sh
kubectl apply -f ./metrics-server
```

Starting Prometheus and Grafana

```sh
kubectl apply -f ./prometheus
kubectl apply -f ./grafana
```

Generate the TLS certificates needed by the Prometheus adapter:

```bash
make certs
```

Start the Prometheus custom metrics API adapter:

```sh
kubectl apply -f ./custom-metrics-api
```

Starting kube-state-metrics

```sh
kubectl apply -f ./kube-state-metrics
```

List the custom metrics provided by Prometheus:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .
```

Get the FS usage for all the pods in the `monitoring` namespace:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/monitoring/pods/*/fs_usage_bytes" | jq .
```

The `kube-znn` app exposes a custom metric named `caddy_http_request_duration_seconds_success_99p`. 

Get the 99th requests from the custom metrics API:

```bash
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/caddy_http_request_duration_seconds_success_99p" | jq .
```
```json
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/%2A/caddy_http_request_duration_seconds_success_99p"
  },
  "items": [
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "podinfo-6b86c8ccc9-kv5g9",
        "apiVersion": "/__internal"
      },
      "metricName": "caddy_http_request_duration_seconds_success_99p",
      "timestamp": "2018-01-10T16:49:07Z",
      "value": "901"
    },
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "default",
        "name": "podinfo-6b86c8ccc9-nm7bl",
        "apiVersion": "/__internal"
      },
      "metricName": "caddy_http_request_duration_seconds_success_99p",
      "timestamp": "2018-01-10T16:49:07Z",
      "value": "898"
    }
  ]
}
```

### Starting Kubow

```sh
kubectl apply -f ./kubow/config
kubectl apply -f ./kubow/kubow-deployment.yaml
```

### Make sure if everything is up and running

```sh
kubectl get all --all-namespaces
```
