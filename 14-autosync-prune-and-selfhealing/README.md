## Auto sync
- By default, argocd will only report as `outofsync` and won;t update the manifests automatically.

- enabling `autosync` allows argocd to update the manifests, without manual approval

## Pruning
- By defaault, argocd won;t delete the manifests from k8s clusters, if it cannot find them in the specified source.
- In other words, if we delete the manifests in source, by default argocd won;t delete them in the cluster.

- enabling `pruning` will allow argocd to delete the orphaned manifests from cluster which it cannot find in the source during sync process.

- Also, when `sync` option enabled, `pruning` will also happen when if pruning too enabled

## self-healing

