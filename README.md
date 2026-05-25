# Flux GitOps Repository

This repository is a Hops Flux GitOps template. It is intended to be used by
`FluxGitopsStack`, which creates the GitHub repository, installs Flux, and wires
Flux `Kustomization` resources to these paths:

```text
apps/        # application manifests and Flux app resources
crossplane/  # optional Crossplane packages and platform resources
```

Both directories include explicit `kustomization.yaml` files with empty
`resources` lists. Add manifests to a directory, then list them in that
directory's `kustomization.yaml`.

## Applications

Plain Kubernetes manifests can live directly under `apps/`.

```yaml
# apps/kustomization.yaml
resources:
- namespace.yaml
- deployment.yaml
- service.yaml
```

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

Then add the file to `apps/kustomization.yaml`.

```yaml
resources:
- ingress-nginx.yaml
```

## Crossplane

`crossplane/` is optional. Enable `spec.kustomizations.crossplane.enabled` on
`FluxGitopsStack` before expecting this path to reconcile.

Use it for Crossplane packages and platform resources:

```yaml
# crossplane/kustomization.yaml
resources:
- configuration.yaml
```

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
- Do not commit secrets. Use External Secrets or another secret management path.
- `FluxGitopsStack` owns the root Flux source and path Kustomizations; only add
  Flux bootstrap resources here if you intentionally want this repository to
  manage Flux itself.
