# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a GitOps repository for managing Kubernetes applications using ArgoCD.

```
.
├── apps/                    # Application manifests organized by service
│   ├── traefik/            # Traefik ingress controller
│   └── monitoring/         # Monitoring stack (Prometheus, Grafana, etc.)
├── argocd-apps/            # ArgoCD Application definitions
│   ├── app-of-apps.yaml   # Root application managing all other apps
│   ├── traefik-app.yaml   # Traefik application definition
│   └── monitoring-app.yaml # Monitoring application definition
└── infrastructure/         # Core infrastructure components
```

## Key Commands

### Cluster Management
- `kubectl get pods -n argocd` - Check ArgoCD status
- `kubectl port-forward svc/argocd-server -n argocd 8080:443` - Access ArgoCD UI
- `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d` - Get ArgoCD admin password

### ArgoCD CLI
- `argocd app sync <app-name>` - Manually sync an application
- `argocd app get <app-name>` - Get application status
- `argocd app list` - List all applications

### Development Workflow
1. Make changes to application manifests in `apps/` directory
2. Commit and push changes to Git repository
3. ArgoCD will automatically detect and sync changes (if auto-sync enabled)
4. For manual sync: use ArgoCD UI or CLI

## Application Management

- Each application has its own directory under `apps/`
- Use Kustomize for configuration management
- ArgoCD Applications are defined in `argocd-apps/`
- The `app-of-apps` pattern is used for managing multiple applications

## Important Notes

- All applications use automated sync with prune and self-heal enabled
- Namespace creation is handled automatically via `CreateNamespace=true`
- Repository URL in ArgoCD applications should match the actual Git repository URL
- ArgoCD runs in the `argocd` namespace