# Grid Station AKS Deployment

This repository contains Kubernetes manifests for deploying a grid station monitoring system on Azure Kubernetes Service (AKS). The idea of design is illustrated in below picture
![image](https://github.com/user-attachments/assets/7db9d176-5117-49f0-b77e-436afcddfc16)


## Repository Structure

```
grid-station/
├── README.md
├── .github/
│   └── workflows/
│       └── ci-cd.yaml
├── argocd/
│   ├── projects/
│   │   └── grid-station-project.yaml
│   └── applications/
│       └── grid-station-app.yaml
└── manifests/
    ├── namespace.yaml
    ├── configmaps/
    │   ├── iec61850-config.yaml
    │   ├── ml-processing-config.yaml
    │   └── opcua-config.yaml
    ├── storage/
    │   ├── iec61850-data-pvc.yaml
    │   └── ml-model-storage-pvc.yaml
    ├── networking/
    │   ├── ec61850-simulator-policy.yaml
    │   ├── ml-processing-policy.yaml
    │   └── scada-interface-policy.yaml
    ├── ec61850-simulator/
    │   ├── deployment.yaml
    │   └── service.yaml
    ├── ml-processing/
    │   ├── deployment.yaml
    │   └── service.yaml
    └── scada-interface/
        ├── deployment.yaml
        └── service.yaml
```

## Setup Instructions

1. Clone this repository
2. Ensure you have access to your AKS cluster
3. Install ArgoCD in your cluster
4. Apply the ArgoCD project and application manifests
5. ArgoCD will automatically deploy the grid station components

## Components

The system consists of three main components:

1. **IEC61850 Simulator Pod**: Simulates an IEC61850 server for electric grid data
2. **ML Processing Pod**: Processes sensor data from the IEC61850 server and generates insights
3. **SCADA Interface Pod**: Exposes the data and insights via OPC UA protocol

## Network Flow

- The IEC61850 Simulator provides sensor data to the ML Processing Pod
- The ML Processing Pod analyzes the data and sends insights to the SCADA Interface
- The SCADA Interface exposes the data and insights via OPC UA to external systems
- Optionally, the ML Processing Pod can send feedback to the IEC61850 Simulator

![image](https://github.com/user-attachments/assets/beda5964-3356-45f6-9029-13b7d5b01416)

## Deployment Architecture

This setup runs on Azure Kubernetes Service (AKS) using the Azure CNI networking plugin. The components communicate within the cluster network and expose services externally as needed.

'''
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

kubectl apply -f argocd/projects/grid-station-project.yaml

kubectl apply -f argocd/applications/grid-station-app.yaml

kubectl apply -f manifests/configmaps/

kubectl apply -f manifests/storage/

kubectl apply -f manifests/networking/

kubectl apply -f manifests/ec61850-simulator/

kubectl apply -f manifests/ml-processing/

kubectl apply -f manifests/scada-interface/

'''
# Port forwarding 
kubectl port-forward svc/argocd-server -n argocd 8080:443

kubectl port-forward svc/ec61850-simulator 8091:8091

# Port forward ML Processing service
# Update with the specific ports you used
kubectl port-forward svc/ml-processing 8092:8092

# Port forward SCADA Interface
kubectl port-forward svc/scada-interface 8093:8093



