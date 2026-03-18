# APP_NAME Knative Service

APP_DESCRIPTION

## Using This Template

Replace the following placeholder values throughout all files before use:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `APP_NAME` | Application name (used as namespace, service name, container name) | `my-app` |
| `APP_IMAGE` | Container image reference | `ghcr.io/my-org/my-app:latest` |
| `APP_PORT` | Container port the app listens on | `8080` |
| `APP_DESCRIPTION` | Short description of the app (this README only) | `HTTP request testing tool` |

```bash
# Replace placeholders (run from the app directory)
find . -type f -name "*.yaml" -o -name "*.md" | xargs sed -i '' \
  -e 's/APP_NAME/my-app/g' \
  -e 's/APP_IMAGE/ghcr.io\/my-org\/my-app:latest/g' \
  -e 's/APP_PORT/8080/g' \
  -e 's/APP_DESCRIPTION/My app description/g'
```

## Structure

- `base/` - Base Knative Service manifests
- `overlays/production/` - Production configuration (higher replica count)
- `overlays/staging/` - Staging configuration (lower replica count)

## Deployment

This app is deployed via Flux CD. To deploy to a cluster instance:

1. Add a GitRepository + Kustomization in your cluster instance repo under `clusters/production/apps/`
2. Point to the appropriate overlay (`overlays/production` or `overlays/staging`)
3. Commit and push - Flux will automatically deploy

Example:

```yaml
# clusters/production/apps/APP_NAME-source.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: app-APP_NAME
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/afloren-dev/app-APP_NAME
  ref:
    branch: main

---
# clusters/production/apps/APP_NAME.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: app-APP_NAME
  namespace: flux-system
spec:
  sourceRef:
    kind: GitRepository
    name: app-APP_NAME
  path: ./overlays/production
  interval: 10m
  prune: true
  dependsOn:
    - name: platform-autoscaling
    - name: platform-mesh
  postBuild:
    substituteFrom:
      - kind: ConfigMap
        name: platform-vars
```

## Configuration

### Base Configuration

- **Min Scale**: 1 replica
- **Max Scale**: 5 replicas
- **Resources**: 100m CPU / 128Mi RAM (request), 500m CPU / 256Mi RAM (limit)

### Production Overlay

- **Min Scale**: 2 replicas
- **Max Scale**: 10 replicas

### Staging Overlay

- **Min Scale**: 1 replica
- **Max Scale**: 3 replicas

## Testing Locally

```bash
# Validate manifests
yamllint .
kubectl kustomize base/ | kubeconform -ignore-missing-schemas
kubectl kustomize overlays/production/ | kubeconform -ignore-missing-schemas

# Apply to local cluster
kubectl apply -k base/
# or
kubectl apply -k overlays/production/
```

## License

MIT
