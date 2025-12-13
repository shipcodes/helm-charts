# Deployment Guide

## Prerequisites

1. **Kubernetes Cluster** (v1.24+)
2. **ArgoCD** installed
3. **Nginx Ingress Controller** installed
4. **Cert-Manager** for SSL certificates
5. **GitHub Container Registry** access

## Step 1: Create Secrets

### Production Secrets

```bash
# Create namespace
kubectl create namespace janeonthegame-prod

# Backend secrets
kubectl create secret generic backend-secrets \
  --namespace=janeonthegame-prod \
  --from-literal=DATABASE_URL="postgresql://user:password@hostname:5432/janeonthegame" \
  --from-literal=JWT_SECRET="your-super-secret-jwt-key-change-this" \
  --from-literal=PAYSTACK_SECRET_KEY="sk_live_your_paystack_key" \
  --from-literal=FOOTBALL_DATA_ORG_KEY="your_api_key_here" \
  --from-literal=AFRICASTALKING_API_KEY="your_api_key" \
  --from-literal=AFRICASTALKING_USERNAME="your_username" \
  --from-literal=SMTP_HOST="smtp.gmail.com" \
  --from-literal=SMTP_PORT="587" \
  --from-literal=SMTP_USER="your-email@gmail.com" \
  --from-literal=SMTP_PASS="your-app-password"

# Frontend secrets
kubectl create secret generic frontend-secrets \
  --namespace=janeonthegame-prod \
  --from-literal=NEXT_PUBLIC_API_URL="https://api.janeonthegame.com/graphql"
```

### Staging Secrets

```bash
# Create namespace
kubectl create namespace janeonthegame-staging

# Backend secrets
kubectl create secret generic backend-staging-secrets \
  --namespace=janeonthegame-staging \
  --from-literal=DATABASE_URL="postgresql://user:password@hostname:5432/janeonthegame_staging" \
  --from-literal=JWT_SECRET="staging-jwt-secret" \
  --from-literal=PAYSTACK_SECRET_KEY="sk_test_your_paystack_test_key" \
  --from-literal=FOOTBALL_DATA_ORG_KEY="your_api_key_here"

# Frontend secrets
kubectl create secret generic frontend-staging-secrets \
  --namespace=janeonthegame-staging \
  --from-literal=NEXT_PUBLIC_API_URL="https://api-staging.janeonthegame.com/graphql"
```

## Step 2: Configure GitHub Container Registry

```bash
# Create Docker registry secret for pulling images
kubectl create secret docker-registry ghcr-secret \
  --namespace=janeonthegame-prod \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_PAT \
  --docker-email=YOUR_EMAIL

# Repeat for staging
kubectl create secret docker-registry ghcr-secret \
  --namespace=janeonthegame-staging \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_PAT \
  --docker-email=YOUR_EMAIL
```

## Step 3: Install ArgoCD Applications

```bash
# Apply ArgoCD project
kubectl apply -f argocd/projects/janeonthegame-project.yaml

# Deploy production applications
kubectl apply -f argocd/applications/backend-production.yaml
kubectl apply -f argocd/applications/frontend-production.yaml

# Deploy staging applications (optional)
kubectl apply -f argocd/applications/backend-staging.yaml
kubectl apply -f argocd/applications/frontend-staging.yaml
```

## Step 4: Verify Deployment

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Check pods
kubectl get pods -n janeonthegame-prod

# Check services
kubectl get svc -n janeonthegame-prod

# Check ingress
kubectl get ingress -n janeonthegame-prod
```

## Step 5: Access ArgoCD UI

```bash
# Port forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access UI at https://localhost:8080
# Username: admin
# Password: (from above command)
```

## CI/CD Workflow

1. **Push code** to GitHub (main or develop branch)
2. **GitHub Actions** builds Docker image and pushes to GHCR
3. **Update values file** in this repo with new image tag (or use `latest`)
4. **ArgoCD** automatically detects change and syncs deployment
5. **Kubernetes** rolls out new version with zero downtime

## Manual Image Update

To deploy a specific version:

```bash
# Edit the values file
vim environments/production/backend-values.yaml

# Change image tag:
# image:
#   tag: "main-abc1234"

# Commit and push
git add environments/production/backend-values.yaml
git commit -m "Deploy backend version abc1234"
git push

# ArgoCD will auto-sync (or manually sync via UI)
```

## Rollback

```bash
# Via ArgoCD UI: Click "History and Rollback"

# Or via CLI:
argocd app rollback backend-production <revision-number>
```

## Monitoring

```bash
# View logs
kubectl logs -f deployment/backend -n janeonthegame-prod
kubectl logs -f deployment/frontend -n janeonthegame-prod

# Describe pod
kubectl describe pod <pod-name> -n janeonthegame-prod

# Execute into pod
kubectl exec -it <pod-name> -n janeonthegame-prod -- sh
```

## Scaling

```bash
# Manual scaling (overrides HPA temporarily)
kubectl scale deployment backend --replicas=5 -n janeonthegame-prod

# HPA will resume control based on CPU/memory metrics
```

## Troubleshooting

### Image Pull Errors
```bash
# Verify secret exists
kubectl get secret ghcr-secret -n janeonthegame-prod

# Add to service account if not auto-configured
kubectl patch serviceaccount default -n janeonthegame-prod \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}'
```

### DNS Issues
```bash
# Verify ingress
kubectl describe ingress -n janeonthegame-prod

# Check cert-manager
kubectl get certificate -n janeonthegame-prod
kubectl describe certificate -n janeonthegame-prod
```

### Pod Crashes
```bash
# Check events
kubectl get events -n janeonthegame-prod --sort-by='.lastTimestamp'

# View previous logs
kubectl logs <pod-name> -n janeonthegame-prod --previous
```

## Production Checklist

- [ ] Database backed up and accessible
- [ ] Secrets created in cluster
- [ ] Domain DNS pointing to cluster LoadBalancer
- [ ] SSL certificates configured
- [ ] GitHub Container Registry access configured
- [ ] ArgoCD applications deployed
- [ ] Health checks passing
- [ ] Monitoring configured
- [ ] Backup strategy in place
- [ ] Disaster recovery plan documented
