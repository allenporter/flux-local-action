# flux-local-action

This Github action runs [flux-local](https://github.com/allenporter/flux-local) to verify
the health of a [Flux](https://fluxcd.io/) cluster in a git repository.

## test action

The `test` action will validate the cluster will build, and can optionally
validate flux `HelmRelease` builds and also verify that all objects pass
kyverno policies (e.g. for determining there are no deprecated api resources
or that ingress objects are valid).

This example will run `flux-local test` against the cluster in `clusters/prod` with
helm release expansion enabled.

```yaml
- uses: allenporter/flux-local-action/test@v1
  with:
    path: clusters/prod
    enable-helm: true
    enable-kyverno: false
```
