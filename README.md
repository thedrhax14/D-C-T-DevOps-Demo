# DevOps Engineer Test Assignment

This repository contains the completed test assignment for the **DevOps Engineer** position at D-C-T Group. It demonstrates setting up a Kubernetes cluster using Minikube, deploying a sample web application, configuring an Ingress, implementing Horizontal Pod Autoscaler (HPA), and integrating Datadog monitoring.

---

## **Table of Contents**
1. [Prerequisites](#prerequisites)  
2. [Cluster Setup](#cluster-setup)  
3. [Deploy Sample Application](#deploy-sample-application)  
4. [Ingress Configuration](#ingress-configuration)  
5. [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler-hpa)  
6. [Datadog Monitoring](#datadog-monitoring)  
7. [Testing & Validation](#testing--validation)  
8. [References](#references)  

---

## **Prerequisites**

- macOS 
- [Docker](https://www.docker.com/) installed  
- [Minikube](https://minikube.sigs.k8s.io/docs/) installed  
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed  
- Datadog account with API key

---

## **Cluster Setup**

1. Start Minikube:

```bash
minikube start --driver=docker
````

2. Enable required addons:

```bash
minikube addons enable ingress
minikube addons enable metrics-server
```

3. Configure `/etc/hosts` (or local equivalent) for the Ingress host:

```text
127.0.0.1 webapp.local
```

---

## **Deploy Sample Application**

1. Apply the deployment and service:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

2. Verify pods and service are running:

```bash
kubectl get pods
kubectl get svc
```

---

## **Ingress Configuration**

1. Apply Ingress resource:

```bash
kubectl apply -f ingress.yaml
```

2. Start Minikube tunnel to expose ports 80/443 (needed on Mac OS):

```bash
sudo minikube tunnel
```

3. Verify Ingress:

```bash
kubectl get ingress
```

Access your app in a browser: `http://webapp.local`

---

## **Horizontal Pod Autoscaler (HPA)**

1. Ensure your deployment has resource requests and limits. For testing maybe better to use less limits to see HPA working in case if you have more resourceful system:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

2. Apply HPA:

```bash
kubectl apply -f hpa.yaml
```

3. Verify HPA:

```bash
kubectl get hpa -w
```

4. Generate load to test scaling:

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
# Inside container:
while true; do wget -q -O- http://webapp-service.default.svc.cluster.local; done
```

---

## **Datadog Monitoring**

1. Install Datadog agent:

```bash
kubectl apply -f datadog-agent.yaml
```

2. Verify agent pods:

```bash
kubectl get pods -n datadog
```

3. Dashboard Metrics (Minikube):

* CPU usage: `container.cpu.usage`
* Memory usage: `container.memory.usage`
* CPU/Memory limits: `container.cpu.limit`, `container.memory.limit`

4. Alerts (Monitors):

* Memory usage > 70%

---

## **Testing & Validation**

1. Access web app: `http://webapp.local`
2. Check HPA scales pods automatically under load
3. Confirm Datadog dashboards update metrics and alerts trigger as expected
4. Capture screenshots for documentation

---

## **References**

* [Minikube Official Docs](https://minikube.sigs.k8s.io/docs/)
* [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [Datadog Kubernetes Monitoring](https://docs.datadoghq.com/agent/kubernetes/)

---

## **Deliverables**

* All YAML manifests: `deployment.yaml`, `service.yaml`, `ingress.yaml`, `hpa.yaml`, `datadog-agent.yaml`
* Screenshots of:

  * **Running pods and service** (`running_pods_and_services.png`) - Shows output `kubectl get pods`, `kubectl get svc` and `kubectl get svc`. Specifically that pods are running and ready, the service exposes them internally, and the Ingress routes webapp.local to the pods via NGINX.
  
  * **HPA scaling** (`hpa_with_simulated_workload.png`) - Demonstrates the HorizontalPodAutoscaler in action with `kubectl get hpa -w` output showing real-time scaling from initial replicas to increased replicas under simulated load, with memory utilization metrics triggering the autoscaling behavior. I by accident triggered the autoscalling it got already scaled ones.
  
  * **Datadog dashboard** (`datadog_dashboard.png`) - Custom Datadog dashboard displaying key Kubernetes metrics including container CPU usage, memory usage and CPU/memory limits for monitoring the Minikube cluster
  
  * **Datadog alerts** (`datadog_simple_custom_alert.png`) - Configured Datadog monitors showing alerts for memory usage > 70%
  
  * **Web application** (`hypothetical_webapp.png`) - Browser view of the nginx webapp accessible at `http://webapp.local` through the Ingress controller, demonstrating successful end-to-end connectivity

---

**Note:** This repository demonstrates a fully functional Minikube setup replicating the requested assignment task. All steps are reproducible on a local machine without AWS access.
