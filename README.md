# Kbot Monitoring Stack

Monitoring stack for the Kbot project deployed to Kubernetes using Flux GitOps.

## Components

- OpenTelemetry Operator
- OpenTelemetry Collector
- Prometheus
- Fluent Bit
- Grafana Loki
- Grafana
- Flux CD

## Architecture

Kbot exposes Prometheus metrics on `/metrics:8080`.

Prometheus scrapes Kbot metrics.

Fluent Bit runs as a DaemonSet and collects Kubernetes container logs and node logs. Logs are exported to Grafana Loki.

Grafana uses Prometheus and Loki as datasources.

OpenTelemetry Collector is deployed and managed by the OpenTelemetry Operator.

## GitOps

The monitoring stack is stored in:

infrastructure/monitoring/

OpenTelemetry resources are stored in:

infrastructure/otel/

Flux Kustomization:

flux-system/kbot-monitoring-stack

All monitoring resources are deployed from Git by Flux.


## Prometheus

Prometheus is deployed using `kube-prometheus-stack`.

Kbot exposes Prometheus metrics on port `8080`.

The application is instrumented with the custom counter `kbot_messages_total`.

The counter is incremented for every text message received by Kbot.

Prometheus scrapes Kbot using the `kbot` job.

The target was verified with Prometheus and returned `up{job="kbot"} = 1`.

## Fluent Bit and Loki

Fluent Bit runs as a Kubernetes DaemonSet.

It collects Kubernetes container logs and node/system logs and exports them to Grafana Loki.

Kbot logs were verified in Loki using the query:

`{job="fluent-bit",source="kubernetes"}`

## OpenTelemetry

OpenTelemetry is deployed using the OpenTelemetry Kubernetes Operator.

The Collector is defined in `infrastructure/otel/collector.yaml`.

The Collector uses OTLP gRPC and HTTP receivers, a batch processor and a debug exporter.

The Collector was verified as managed by the OpenTelemetry Operator.

## Grafana

Grafana is deployed through Flux.

Configured datasources:

- Prometheus
- Loki

### Demo dashboard

The repository contains the provisioned **Kbot Monitoring** dashboard.

Dashboard UID: `kbot-monitoring`

Folder: `Monitoring`

The dashboard contains:

- Kbot Availability
- Kbot Messages Total
- Kbot Message Rate
- Kubernetes Logs

Dashboard source:

`infrastructure/monitoring/dashboard.json`

Dashboard ConfigMap:

`infrastructure/monitoring/grafana-dashboard-configmap.yaml`

The dashboard was verified through the Grafana API.

## Verification

The main Kubernetes resources can be checked with:

    kubectl get kustomization kbot-monitoring-stack -n flux-system
    kubectl get helmrelease prometheus -n flux-system
    kubectl get helmrelease loki -n flux-system
    kubectl get daemonset -n monitoring monitoring-fluent-bit
    kubectl get helmrelease grafana -n flux-system
    kubectl get opentelemetrycollector -n monitoring

## Result

The monitoring stack is deployed in Kubernetes using Flux GitOps.

The project is instrumented for Prometheus metrics.

Fluent Bit collects project and node logs and exports them to Loki.

OpenTelemetry is deployed and managed by the Kubernetes Operator.

Grafana provides a provisioned dashboard combining application metrics and Kubernetes logs.
