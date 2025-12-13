# Jane on the Game - Kubernetes Deployment

This repository contains Helm charts and ArgoCD configurations for deploying the Jane on the Game betting platform to Kubernetes.

## Repository Structure

```
.
├── charts/
│   ├── backend/          # Backend (NestJS API) Helm chart
│   └── frontend/         # Frontend (Next.js) Helm chart
├── argocd/
│   ├── applications/     # ArgoCD Application manifests
│   └── projects/         # ArgoCD Project definitions
└── environments/
    ├── dev/             # Development environment values
    ├── staging/         # Staging environment values
    └── production/      # Production environment values
```

## Prerequisites

- Kubernetes cluster (v1.24+)
- Helm 3.x
- ArgoCD installed on cluster
- kubectl configured

## Quick Start

### 1. Install with Helm (Manual)

```bash
# Backend
helm install backend ./charts/backend -f environments/production/backend-values.yaml

# Frontend
helm install frontend ./charts/frontend -f environments/production/frontend-values.yaml
```

### 2. Deploy with ArgoCD (Recommended)

```bash
# Apply ArgoCD applications
kubectl apply -f argocd/applications/
```

## Environment Variables

Configure secrets in your cluster before deployment:

```bash
# Backend secrets
kubectl create secret generic backend-secrets \
  --from-literal=DATABASE_URL="postgresql://user:pass@host:5432/db" \
  --from-literal=JWT_SECRET="your-jwt-secret" \
  --from-literal=PAYSTACK_SECRET_KEY="your-paystack-key" \
  --from-literal=FOOTBALL_DATA_ORG_KEY="your-api-key"

# Frontend secrets
kubectl create secret generic frontend-secrets \
  --from-literal=NEXT_PUBLIC_API_URL="https://api.yourdomain.com/graphql"
```

## Monitoring

- Prometheus metrics exposed on `/metrics`
- Health checks available on `/health`
- Logs aggregated via your cluster's logging solution

## Support

For issues, contact the development team.
