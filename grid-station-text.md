# Grid Station Automation System
## Core Components and Data Flow

### External Components

**Physical Grid Station**
- Power Equipment
- Transformers, Breakers
- Sensors

**External Systems**
- SCADA/HMI
- Control Room
- OPC UA Clients

**ArgoCD**
- GitOps Deployment
- Manifest Synchronization

### AKS Cluster Components

**IEC61850 Simulator** (Port: 102)
- Simulates grid station equipment
- Generates time-series data
- Source of all electrical measurements
- Responds to control commands

**Prometheus Exporter** (Port: 9100)
- Runs alongside the IEC61850 Simulator
- Reads CSV data files
- Converts measurements to Prometheus metrics
- Exposes metrics endpoint

**ML Processing** (Port: 8001)
- Analyzes grid data
- Detects anomalies
- Predicts future load
- Monitors equipment health
- Provides web dashboard

**Decision Engine** (Port: 8002)
- Makes control decisions based on ML analysis
- Implements safety validation
- Automates grid control actions
- Provides API for manual control

**SCADA Interface** (Port: 4840)
- Implements OPC UA server
- Exposes data to industrial systems
- Receives control commands from Decision Engine
- Forwards controls to IEC61850 Simulator

**Storage (PVCs)**
- iec61850-data (1Gi): Time-series measurements
- ml-model-storage (5Gi): ML models & data

**Monitoring Stack**
- Prometheus: Collects & stores metrics
- Grafana (Port: 3000): Grid Station Dashboard
- Alerts & Visualizations

## Data Flow Diagram

```
┌─────────────────┐        ┌───────────────┐        ┌──────────────────┐
│  Physical Grid  │        │    ArgoCD     │        │  External Systems │
│    Station      │        │  GitOps Depl. │        │    SCADA/HMI     │
└────────┬────────┘        └───────┬───────┘        └─────────┬────────┘
         │                         │                          │
         ▼                         ▼                          │
┌─────────────────────────────────────────────────────────────┴──────────┐
│                      AKS CLUSTER                                        │
│                                                                         │
│  ┌─────────────┐       ┌────────────┐       ┌─────────────┐            │
│  │  IEC61850   │       │    ML      │       │  Decision   │            │
│  │ Simulator   │──────▶│ Processing │──────▶│   Engine    │            │
│  │  (102)      │       │  (8001)    │       │   (8002)    │            │
│  └──────┬──────┘       └──────┬─────┘       └──────┬──────┘            │
│         │                     │                    │                    │
│         │                     │                    │                    │
│         ▼                     ▼                    ▼                    │
│  ┌─────────────┐       ┌────────────┐       ┌─────────────┐            │
│  │ Prometheus  │       │            │       │   SCADA     │            │
│  │  Exporter   │──────▶│  Storage   │◀──────│ Interface   │────────────┘
│  │   (9100)    │       │   (PVCs)   │       │   (4840)    │
│  └──────┬──────┘       └────────────┘       └─────────────┘
│         │                                                               
│         ▼                                                               
│  ┌─────────────┐                                                        
│  │ Monitoring  │                                                        
│  │   Stack     │                                                        
│  │   (3000)    │                                                        
│  └─────────────┘                                                        
│                                                                         
└─────────────────────────────────────────────────────────────────────────┘
```

## Key Data Flows

1. **Grid Data Collection**
   - Physical Grid Station → IEC61850 Simulator
   - IEC61850 Simulator → ML Processing
   - IEC61850 Simulator → Prometheus Exporter → Monitoring Stack

2. **Analysis and Decision Making**
   - ML Processing → Decision Engine
   - Decision Engine → SCADA Interface
   - SCADA Interface → IEC61850 Simulator (control loop)

3. **External Interfaces**
   - SCADA Interface → External Systems (OPC UA)
   - Monitoring Stack → External Systems (dashboards)
   - ArgoCD → AKS Cluster (deployments)

4. **Storage**
   - IEC61850 Simulator ↔ Storage
   - ML Processing ↔ Storage

## Port Summary

| Component             | Port | Protocol  | Purpose                            |
|-----------------------|------|-----------|----------------------------------- |
| IEC61850 Simulator    | 102  | IEC61850  | Grid station simulation            |
| ML Processing         | 8001 | HTTP      | Analysis API and web dashboard     |
| Decision Engine       | 8002 | HTTP      | Control API                        |
| SCADA Interface       | 4840 | OPC UA    | Industrial protocol interface      |
| Prometheus Exporter   | 9100 | HTTP      | Metrics endpoint                   |
| Grafana               | 3000 | HTTP      | Monitoring dashboards              |
