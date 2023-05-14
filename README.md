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
- uses: allenporter/flux-local-action/test@v2
  with:
    path: clusters/prod
    enable-helm: true
    enable-kyverno: false
```

## diff action

The `diff` action will show you the final diffs of `Kustomization` or `HelmRelease`
objects that are fully built. While typically you can just read diffs to understand
how kustomzations may be affected, this action also supports overlays and multiple
clusters showing you the final output.

This is an example that diffs a `HelmRelease`:

```yaml
- uses: allenporter/flux-local-action/diff@v2
  id: diff
  with:
    path: clusters/prod
    resource: helmrelease
- name: PR Comments
  uses: mshick/add-pr-comment@v2
  if: ${{ steps.diff.outputs.diff != '' }}
  with:
    repo-token: ${{ secrets.GITHUB_TOKEN }}
    message-failure: Unable to post diff
    message: |
      `````diff
      ${{ steps.diff.outputs.diff }}
      `````
```

This is an example of a workflow that will diff `Kustomization` and `HelmRelease` objects
in a repo with multiple clusters (`dev` and `prod`):

```yaml
jobs:
  diffs:
    name: Compute diffs
    runs-on: ubuntu-latest
    strategy:
      matrix:
        cluster_path:
          - clusters/dev
          - clusters/prod
        resource:
          - helmrelease
          - kustomization
    steps:
      - uses: allenporter/flux-local-action/diff@v2
        id: diff
        with:
          path: ${{ matrix.cluster_path }}
          resource: ${{ matrix.resource }}
      - name: PR Comments
        uses: mshick/add-pr-comment@v2
        if: ${{ steps.diff.outputs.diff != '' }}
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          message-id: ${{ matrix.cluster_path }}/${{ matrix.resource }}
          message-failure: Unable to post kustomization diff
          message: |
            `````diff
            ${{ steps.diff.outputs.diff }}
            `````
```
