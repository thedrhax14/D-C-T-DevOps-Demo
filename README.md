# Dream Come True DevOps Demo

## Overview
This project demonstrates Kubernetes and monitoring capabilities using Minikube to simulate an Amazon EKS environment.

## Components
- **Kubernetes Cluster**: Minikube-based local cluster simulating EKS
- **Sample Application**: Deployed application for demonstration
- **Ingress Controller**: Traffic routing and management
- **Horizontal Pod Autoscaler (HPA)**: Automatic scaling based on metrics
- **Datadog Integration**: Monitoring and observability

## Project Structure
```
.
├── deployment.yaml              # Application deployment configuration
├── service.yaml                 # Service configuration
├── ingress.yaml                 # Ingress controller configuration
├── hpa.yaml                     # Horizontal Pod Autoscaler configuration
├── datadog-agent.yaml.example   # Datadog agent template (copy and add your API key)
├── screenshots/                 # Documentation screenshots
├── .gitignore                   # Git ignore rules
└── README.md                    # Project documentation
```

## Setup Instructions

### Security Note
**Important**: The actual `datadog-agent.yaml` file with your API key is excluded from git via `.gitignore`.
To set up Datadog monitoring:
1. Copy `datadog-agent.yaml.example` to `datadog-agent.yaml`
2. Replace `YOUR_DATADOG_API_KEY_HERE` with your actual Datadog API key
3. Apply the configuration: `kubectl apply -f datadog-agent.yaml`

(Additional setup instructions to be documented)

## Deliverables
(To be documented)

