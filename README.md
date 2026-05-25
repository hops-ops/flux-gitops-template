# Flux GitOps Repository

This repository is a Hops Flux GitOps template. It is intended to be used by
`FluxGitopsStack`, which creates the GitHub repository, installs Flux, and wires
Flux `Kustomization` resources to these paths:

```text
apps/        # application manifests and Flux app resources
crossplane/  # optional Crossplane packages and platform resources
```

Flux auto-generates the Kustomize input for these directories when no
`kustomization.yaml` file is present. Add Kubernetes YAML files under a watched
directory and Flux applies them. The `.gitkeep` files only keep the empty
directories present in Git; Flux ignores them.

## Applications

Plain Kubernetes manifests can live directly under `apps/`.

For Helm charts, commit Flux `HelmRepository` and `HelmRelease` resources.

```yaml
# apps/ingress-nginx.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: ingress-nginx
  namespace: flux-system
spec:
  interval: 1h
  url: https://kubernetes.github.io/ingress-nginx
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: ingress-nginx
  namespace: flux-system
spec:
  interval: 10m
  targetNamespace: ingress-nginx
  install:
    createNamespace: true
  chart:
    spec:
      chart: ingress-nginx
      version: 4.12.1
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: flux-system
  values:
    controller:
      replicaCount: 2
```

Once committed under `apps/`, Flux reconciles the file automatically.

## Crossplane

`crossplane/` is optional. Enable `spec.kustomizations.crossplane.enabled` on
`FluxGitopsStack` before expecting this path to reconcile.

Use it for Crossplane packages and platform resources:

Example package manifest:

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Configuration
metadata:
  name: platform-foundation
spec:
  package: ghcr.io/hops-ops/platform-foundation:v0.1.0
```

## Notes

- Manifests should include their intended namespace unless they are
  cluster-scoped.
- Every YAML file under `apps/` and enabled `crossplane/` paths must be a valid
  Kubernetes manifest. Keep examples, values files, and notes outside those
  paths, or add a `kustomization.yaml` if you intentionally want explicit
  resource selection.
- Do not commit secrets. Use External Secrets or another secret management path.
- `FluxGitopsStack` owns the root Flux source and path Kustomizations; only add
  Flux bootstrap resources here if you intentionally want this repository to
  manage Flux itself.
