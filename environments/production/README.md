# Production Environment Secrets

## ⚠️ IMPORTANT: Never commit real secrets to Git!

These secret files contain **PLACEHOLDER** values. Before deploying:

### 1. Update Secret Values

Edit each secret file and replace placeholder values with real ones:

- `backend-secrets.yaml` - Update database URL, JWT secret, Paystack keys, etc.
- `frontend-secrets.yaml` - Update API URL if different
- `ghcr-secret.yaml` - Generate real Docker registry secret (see below)

### 2. Generate GHCR Secret

Run this command to generate the real Docker registry secret:

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=janeonthegame \
  --docker-password=YOUR_GITHUB_PERSONAL_ACCESS_TOKEN \
  --docker-email=your-email@example.com \
  --namespace=default \
  --dry-run=client -o yaml > ghcr-secret.yaml
```

### 3. Apply Secrets

```bash
# Apply all secrets at once
kubectl apply -f backend-secrets.yaml
kubectl apply -f frontend-secrets.yaml
kubectl apply -f ghcr-secret.yaml
```

### 4. Verify Secrets

```bash
kubectl get secrets -n default
kubectl describe secret backend-secrets -n default
kubectl describe secret frontend-secrets -n default
```

## Security Best Practices

1. **Never commit real secrets** - Add `*-secrets.yaml` to `.gitignore` if needed
2. **Use separate secrets per environment** - dev, staging, production
3. **Rotate secrets regularly** - Especially JWT secrets and API keys
4. **Use strong passwords** - Min 32 characters for JWT_SECRET
5. **Enable RBAC** - Restrict who can view/edit secrets in your cluster

## Alternative: Sealed Secrets

For GitOps with secrets in Git, consider using:
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [External Secrets Operator](https://external-secrets.io/)
- Cloud provider secret managers (AWS Secrets Manager, Google Secret Manager, etc.)
