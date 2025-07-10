# Gitops With ArgoCD

## 🗂️ Structure Overview
```
gitops-argocd/
├── apps/
│   ├── dev/
│   │   ├── my-app/
│   │   │   ├── application.yaml      # Argo CD Application CR for dev
│   │   │   └── values.yaml           # Symlink or copy of values-dev.yaml
│   ├── stg/
│   │   ├── my-app/
│   │   │   ├── application.yaml
│   │   │   └── values.yaml
│   └── prod/
│       ├── my-app/
│       │   ├── application.yaml
│       │   └── values.yaml
├── charts/
│   └── my-app-chart/                 # Reusable Helm chart
│       ├── Chart.yaml
│       ├── templates/
│       └── values-dev.yaml
│       └── values-stg.yaml
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
    syncOptions:
      - CreateNamespace=true
```

### 🧩 Chart Highlights (my-app-chart/)

- Health checks, resource limits, and HPA (deployment.yaml, hpa.yaml)

- External ingress support for ALB (ingress.yaml)

- Secrets and configmaps per environment

- _helpers.tpl to keep naming consistent
