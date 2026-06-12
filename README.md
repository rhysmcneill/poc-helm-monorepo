# helm-monorepo

A monorepo of Helm charts. Each chart is versioned independently using conventional commits and published to the GHCR OCI registry, where it can be referenced by any GitOps process.

## POC scope

This repo is an end-to-end proof of concept covering the full platform delivery pipeline:

```
Helm chart (monorepo)
  → GitHub Actions (conventional commit versioning + OCI publish)
    → GHCR OCI registry
      → ArgoCD (App of Apps + ApplicationSets for multi-cluster)
        → Kargo (environment promotion pipeline)
```

## Charts

| Chart | Description |
|-------|-------------|
| `observability` | Grafana, Loki, Tempo, Prometheus |

## Versioning

Charts are versioned automatically by CI using [Conventional Commits](https://www.conventionalcommits.org/):

| Commit prefix | Version bump |
|---------------|--------------|
| `fix:` | patch — `0.1.0 → 0.1.1` |
| `feat:` | minor — `0.1.0 → 0.2.0` |
| `feat!:` / `BREAKING CHANGE` | major — `0.1.0 → 1.0.0` |

Commits that don't match any prefix produce no release. Each chart is versioned independently — a commit touching only `charts/observability` will only bump the `observability` chart.

## OCI registry

Charts are published to GHCR on every release:

```
oci://ghcr.io/<org>/helm-charts/<chart>:<version>
```

Example:

```
oci://ghcr.io/<org>/helm-charts/observability:0.2.0
```

## ArgoCD reference

Reference a chart directly from the OCI registry in an ArgoCD `Application`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: observability
  namespace: argocd
spec:
  destination:
    namespace: monitoring
    server: https://kubernetes.default.svc
  project: default
  source:
    repoURL: oci://ghcr.io/<org>/helm-charts
    chart: observability
    targetRevision: 0.2.0
    helm:
      valuesObject: {}  # cluster-specific overrides go here
```

## Adding a new chart

1. Create the chart under `charts/<name>/`.
2. Add `<name>` to the `matrix.chart` list in `.github/workflows/release-charts.yml`.
3. Merge a `feat:` commit touching the new chart directory — CI handles the rest.

## Local development

```bash
# Lint a chart
helm lint charts/observability

# Update dependencies
cd charts/observability && helm dependency update .

# Dry-run render
helm template observability charts/observability
```
