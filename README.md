# Gitops With ArgoCD

[![Helm](https://img.shields.io/badge/Helm-Package%20Manager-0F1689?logo=helm&logoColor=white)](https://helm.sh/)
[![Argo CD](https://img.shields.io/badge/Argo%20CD-GitOps%20CD-EF7B4D?logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![GitOps](https://img.shields.io/badge/GitOps-Automated%20Delivery-1F6FEB?logo=git&logoColor=white)](https://opengitops.dev/)


This repository implements a GitOps workflow using Argo CD and Helm to manage Kubernetes deployments across multiple environments: `dev`, `stg` (staging), and `prod`. It is structured to promote environment-specific configuration and reusable Helm charts.
 

## 🗂️ Structure Overview
```
gitops-argocd/
├── apps/
│   ├── dev/
│   │   ├── my-app/
│   │   │   └── application.yaml      # Argo CD Application CR for dev
│   ├── stg/
│   │   ├── my-app/
│   │   │   └── application.yaml
│   └── prod/
│       ├── my-app/
│       │   └── application.yaml
├── charts/
│   └── my-app-chart/                 # Reusable Helm chart
│       ├── Chart.yaml
│       ├── templates/
│       │   ├── frontend-deployment.yaml
│       │   ├── frontend-service.yaml
│       │   ├── backend-deployment.yaml
│       │   ├── backend-service.yaml
│       │   ├── secert.yaml
│       │   └── ingress.yaml            
│       ├── values-dev.yaml
│       ├── values-stg.yaml
│       └── values-prod.yaml
└── README.md
```

## 🚀 How It Works

- Argo CD watches this Git repo and syncs apps based on application.yaml definitions.

- Each environment (dev, stg, prod) has:

  - A dedicated namespace

  - Its own Helm value overrides

  - Independent sync control

- Helm chart is reused across all environments for consistency.

## 🔧 Configuration

Sample Argo CD Application (`apps/dev/my-app/application.yaml`)
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-dev
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/gitops-repo
    targetRevision: HEAD
    path: charts/my-app-chart
    helm:
      valueFiles:
        - values-dev.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
  automated:
    prune: true
    selfHeal: true
    allowEmpty: false
  retry:
    limit: 5
    backoff:
      duration: 5s
      factor: 2
      maxDuration: 3m
  syncOptions:
    - CreateNamespace=true
    - Validate=true
    - ApplyOutOfSyncOnly=true
```

## 🧩 Chart Highlights (my-app-chart/)

- Health checks, resource limits, and HPA (deployment.yaml, hpa.yaml)

- External ingress support for ALB (ingress.yaml)

- Secrets and configmaps per environment

- _helpers.tpl to keep naming consistent

## 📚 Environments

| Environment             | Helm Values File                 | Path                                                 |
|-------------------------|----------------------------------|------------------------------------------------------|
| Dev                     | `values-dev.yaml`                | `apps/dev/my-app/`                                   |
| Staging                 | `values-stg.yaml`                | `apps/stg/my-app/`                                   |
| Production              | `values-prod.yaml`               | `apps/prod/my-app/`                                  |

## 📌 Getting Started

- Install Argo CD

- Push GitOps repo to GitHub

- Apply `application.yaml` via Argo CD CLI or UI:
```bash
argocd app create -f apps/dev/my-app/application.yaml
```
- Watch it sync and deploy into cluster!

