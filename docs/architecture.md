# Architecture Notes

## Components

- **AWS EC2 / Ubuntu:** host environment for Docker and the Kind cluster.
- **Docker:** container runtime used by Kind nodes.
- **Kind:** local Kubernetes cluster running on the EC2 host.
- **kube-prometheus-stack:** Helm chart that deploys the monitoring components and supporting exporters/operators.
- **Prometheus:** scrapes configured metrics targets and evaluates alert rules.
- **Grafana:** queries Prometheus and presents metrics in dashboards.
- **Alertmanager:** receives alerts from Prometheus and applies configured grouping, silencing, and routing behavior.
- **PrometheusRule:** Kubernetes custom resource containing the lab's alert definitions.
- **SSH local port forwarding:** lets a workstation reach web UIs through an SSH connection instead of exposing monitoring ports publicly.

## Data and alert flow

1. Kubernetes components and exporters expose metrics.
2. Prometheus discovers and scrapes configured targets.
3. Prometheus evaluates the `PrometheusRule` definitions.
4. Firing alerts are sent to Alertmanager.
5. Grafana queries Prometheus to visualize metrics.
6. The operator accesses the web UIs through an SSH tunnel.
