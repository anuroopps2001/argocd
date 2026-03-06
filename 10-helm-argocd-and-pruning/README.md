* argoCD will take the helm charts and generate the k8s manifest files using
```bash
helm template <chart> --values ....
```
**Argo CD uses Helm only as a template renderer, not as a release manager.**

* When we the resources created with one repo source/path and if we change the path afterwards, argocd allows us to prune the resources before syncing based on the resources absent from the path specified inside the application CRD yaml.

When Argo CD deploys a Helm chart, it effectively runs:
```bash
helm template <chart>
```
This command:

- renders the Helm chart

- produces plain Kubernetes manifests

Example output:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
```

Argo CD then applies those manifests directly to the cluster using the Kubernetes API.

So the real flow is:
```bash
Git Repo (Helm chart)
        │
        ▼
Argo CD runs: helm template
        │
        ▼
Generated Kubernetes YAML
        │
        ▼
Applied to cluster
```

Because helm install was never executed, there is no Helm release stored in the cluster.

Therefore:
```bash
helm list
```
returns:
```bash
No releases found
```
This is expected behavior.


But since Argo CD never created a Helm release, those records do not exist.

Instead, Argo CD tracks application state using its own resources.

The real source of truth becomes:
```bash
Argo CD Application CR
+
Git repository
```
