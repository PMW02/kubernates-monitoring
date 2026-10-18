# Deployment and Testing Runbook

## 1. Inspect the environment

```bash
kind get clusters
kubectl cluster-info
helm list -n monitoring
kubectl get pods,svc -n monitoring
```

## 2. Install if not already installed

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

If the namespace already exists, `kubectl create namespace monitoring` may report that it exists; inspect the release before proceeding. Do not reinstall an existing release without checking its state.

## 3. Apply the custom alert rules

```bash
kubectl apply -f k8s/demo-alerts.yaml
kubectl get prometheusrule demo-alerts -n monitoring
```

In Prometheus, open **Alerts** and confirm `DemoAlwaysFiring` is firing. In Alertmanager, confirm the alert is visible. This validates the test alert's path; it does not validate email delivery.

## 4. Optional deployment availability test

```bash
kubectl get deployment nginx -n default
```

If it does not exist, create it:

```bash
kubectl create deployment nginx --image=nginx:stable
kubectl rollout status deployment/nginx -n default
```

Then test the alert:

```bash
kubectl scale deployment/nginx --replicas=0 -n default
```

Wait for the one-minute alert duration and confirm `NginxDeploymentUnavailable` fires in Prometheus and appears in Alertmanager. Restore the workload:

```bash
kubectl scale deployment/nginx --replicas=1 -n default
kubectl rollout status deployment/nginx -n default
```

If the alert does not fire, check that the metric exists, that the `namespace` and `deployment` labels match, and that the PrometheusRule is selected by the installed Prometheus instance.

## 5. Access UIs through port-forwarding

On the EC2 host, run each in a separate session:

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```
