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

```
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

```

# Advanced implementation of monitoring and generate data simulation installation
```
helm repo update
kubectl create namespace monitoring

helm install prometheus prometheus-community/kube-prometheus-stack \\n  -n monitoring \\n  -f manifests/monitoring/prometheus-values.yaml
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -n monitoring \\n  --set persistence.enabled=true \\n  --set persistence.size=1Gi \\n  --set service.type=ClusterIP

kubectl get secret -n monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode

kubectl port-forward -n monitoring svc/grafana 3000:80 &
 
kubectl apply -f manifests/configmap/iec61850-data-generator.yaml
kubectl apply -f manifests/configmap/prometheus-exporter.yaml
kubectl apply -f manifests/services/prometheus-exporter.yaml
kubectl apply -f manifests/ec61850-simulator/deployment.yaml
kubectl apply -f manifests/monitoring/service-monitor.yaml
```

# Do port forward when above manifests are done
kubectl port-forward -n grid-station svc/ml-processing 8001:8001 &

kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80 &

kubectl port-forward -n monitoring svc/prometheus-server 9090:9090 &

kubectl port-forward -n grid-station svc/scada-interface 4840:4840 &

Grafana dashboard password could be retrieved using either of
kubectl get secret -n monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
or
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

Grafana dashboard with admin password when used we can get link of grid dashboard

![image](https://github.com/user-attachments/assets/c4dff158-b67a-4463-b422-4a66c2ba4095)

http://127.0.0.1:3000/d/grid-station/grid-station-dashboard?orgId=1&from=now-1h&to=now&timezone=browser&refresh=5s

http://127.0.0.1:8001/ displays ML dashboard

opc.tcp://127.0.0.1:4840 (This cannot be viewed in browser it requires  OPC UA client


