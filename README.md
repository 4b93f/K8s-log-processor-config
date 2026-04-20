# k8s-prod-config

GitOps config repo for [k8s-prod](https://github.com/4b93f-organization/k8s-prod). Managed by ArgoCD — push here, cluster updates automatically.

## Structure

```
argocd/
  application.yaml   # ArgoCD app — deploys the Helm chart
  monitor.yaml       # ArgoCD app — deploys kube-prometheus-stack
chart/
  Chart.yaml
  values.yaml        # All config lives here
  templates/
    api.yaml                  # API Deployment
    api-service.yaml          # API Service (NodePort)
    worker-deployment.yaml    # Worker Deployment
    worker-service.yaml       # Worker metrics Service
    worker-servicemonitor.yaml # Prometheus ServiceMonitor
    namespace.yaml
grafana/
  worker-dashboard.json  # Grafana dashboard for worker metrics
```

## How it works

ArgoCD watches this repo. Any change to `chart/` is automatically synced to the cluster (selfHeal + prune enabled).

```
git push → ArgoCD detects change → helm upgrade → cluster updated
```

## Configuration

All values are in `chart/values.yaml` — images, replicas, resources, env vars. No hardcoding in templates.

To change the API image:
```yaml
# chart/values.yaml
api:
  image: ghcr.io/<org>/k8s-prod-api:<tag>
```

## Deploying ArgoCD Applications

```bash
# Deploy the app
kubectl apply -f argocd/application.yaml

# Deploy monitoring (kube-prometheus-stack)
kubectl apply -f argocd/monitor.yaml
```

## Monitoring

`monitor.yaml` installs `kube-prometheus-stack` (Prometheus + Grafana + Alertmanager).

The `worker-servicemonitor.yaml` auto-discovers worker pods exposing metrics on port 8000 with the label `scrape: "true"`.

Import `grafana/worker-dashboard.json` in Grafana to get the full worker dashboard:
- Messages processed/failed rate
- OCR success/failure totals
- HTTP errors found in logs
- Processing duration

## CI

GitHub Actions runs `helm lint` + `helm template` on every push to validate the chart renders correctly.
