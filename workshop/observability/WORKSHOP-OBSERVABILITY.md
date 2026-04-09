# Observability — Grafana + Prometheus

Monitor your PaperMC server with Prometheus metrics and a Grafana dashboard.

## Prerequisites

Make sure you have completed the main workshop setup and the Metrics section. Your PaperMC server should be running with the Prometheus exporter plugin.

## Install kube-prometheus-stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set grafana.sidecar.dashboards.enabled=true \
  --set grafana.sidecar.dashboards.label=grafana_dashboard
```

Wait for it to come up:

```bash
kubectl get pods -n monitoring
```

## Apply the observability resources

Edit the following files and replace `<YOUR_NAME>` with your name:

- `workshop/observability/certificate.yaml`
- `workshop/observability/httproute.yaml`

Then apply everything:

```bash
kubectl apply -f workshop/observability/servicemonitor.yaml
kubectl apply -f workshop/observability/dashboard.yaml
kubectl apply -f workshop/observability/certificate.yaml
kubectl apply -f workshop/observability/httproute.yaml
```

## Add the Grafana listener to your Gateway

Copy the updated gateway file that includes the Grafana listener:

```bash
cp workshop/observability/gateway-patch.yaml k8s/gateway.yaml
```

Edit `k8s/gateway.yaml` and replace `<YOUR_NAME>` in both the `argocd-https` and `grafana-https` listener hostnames.

Commit and push. ArgoCD will sync the updated gateway.

## Create the DNS record

Get your Gateway IP:

```bash
kubectl get gateway paper-gateway -n paper
```

```bash
doctl compute domain records create cmute.cloud \
  --record-type A \
  --record-name "grafana.<YOUR_NAME>.mc.labs" \
  --record-data <GATEWAY_EXTERNAL_IP> \
  --record-ttl 300
```

## Access Grafana

Get your Grafana password:

```bash
kubectl get secret -n monitoring prometheus-stack-grafana \
  -o jsonpath='{.data.admin-password}' | base64 -d && echo
```

Navigate to `https://grafana.<YOUR_NAME>.mc.labs.cmute.cloud`

- **Username:** admin
- **Password:** output from above

The PaperMC Server dashboard will be available under Dashboards. It shows TPS, tick duration, JVM memory, player count, entities, and more.