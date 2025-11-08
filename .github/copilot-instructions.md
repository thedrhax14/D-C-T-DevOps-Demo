# Copilot Instructions for Dream Come True DevOps Demo

## Project Overview

This is a **Minikube-based Kubernetes demonstration** for a DevOps Engineer test assignment. The project showcases:
- Local Kubernetes cluster setup with Minikube
- Nginx webapp deployment with HPA (Horizontal Pod Autoscaler)
- Ingress configuration for local domain routing (`webapp.local`)
- Datadog monitoring integration via DaemonSet

**Core principle:** Everything runs locally on Minikube without requiring cloud infrastructure.

## Architecture & Components

### Application Stack
- **Deployment:** Simple nginx webapp (`deployment.yaml`) with 2 initial replicas
- **Service:** ClusterIP service (`webapp-service`) exposing port 80
- **Ingress:** Routes `webapp.local` traffic to the service (requires Minikube tunnel)
- **HPA:** Scales 1-5 replicas based on 50% memory utilization threshold
- **Monitoring:** Datadog DaemonSet in dedicated `datadog` namespace

### Key Dependencies
- Minikube addons: `ingress` and `metrics-server` (required for HPA)
- `/etc/hosts` entry: `127.0.0.1 webapp.local` for local DNS
- `minikube tunnel` must run for Ingress to be accessible

## Critical Workflows

### Initial Setup
```bash
# Start cluster and enable required features
minikube start --driver=docker
minikube addons enable ingress
minikube addons enable metrics-server

# Deploy stack in order
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f hpa.yaml
kubectl apply -f datadog-agent.yaml  # Use .example as template
```

### Local Access Pattern
Always run `sudo minikube tunnel` in a separate terminal before accessing `http://webapp.local`. Without the tunnel, Ingress won't route traffic.

### Testing HPA
```bash
# Generate load to trigger scaling
kubectl run -i --tty load-generator --image=busybox /bin/sh
# Inside container:
while true; do wget -q -O- http://webapp-service.default.svc.cluster.local; done
```

Watch HPA response: `kubectl get hpa -w`

## Project-Specific Conventions

### Resource Constraints
All deployments must specify resource requests/limits. This is **mandatory** for HPA functionality:
```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

### Datadog Configuration Pattern
- Use `datadog-agent.yaml.example` as the template (contains placeholder API key)
- Actual `datadog-agent.yaml` contains real credentials and is **gitignored**
- DaemonSet pattern mounts `/var/run/docker.sock` for container metrics
- Always deploys to dedicated `datadog` namespace

### Naming Conventions
- Service: `webapp-service` (not `webapp`)
- Deployment: `webapp`
- Ingress: `webapp-ingress`
- HPA: `webapp-hpa`

## Key Files & Purpose

- `deployment.yaml` - Nginx app with resource constraints for HPA
- `service.yaml` - ClusterIP service (no LoadBalancer/NodePort needed)
- `ingress.yaml` - Routes `webapp.local` hostname to service
- `hpa.yaml` - Memory-based autoscaling (50% threshold)
- `datadog-agent.yaml.example` - Template for monitoring setup
- `screenshots/` - Validation evidence (HPA scaling, Datadog dashboards)

## Important Gotchas

1. **HPA requires metrics-server:** Won't work without the addon enabled
2. **Datadog API keys:** Never commit real keys; use `.example` pattern
3. **Minikube tunnel:** Requires sudo and must stay running for Ingress access
4. **HPA metrics:** Uses `autoscaling/v2` API (not v1) with memory utilization target
5. **Docker socket mount:** Datadog DaemonSet needs host Docker socket access for metrics

## Testing & Validation Checklist

- [ ] Pods running: `kubectl get pods` shows webapp replicas
- [ ] Service accessible internally: `kubectl get svc webapp-service`
- [ ] Ingress routes traffic: `curl http://webapp.local` (with tunnel running)
- [ ] HPA responds to load: Replicas increase under simulated traffic
- [ ] Datadog agent running: `kubectl get pods -n datadog`
- [ ] Screenshots captured in `screenshots/` for deliverables
