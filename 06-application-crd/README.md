### Application CRD YAML structure
```bash
spec.project
```
This is the project for argocd in which, it will manage multiple argocd deployed k8s manifests.

```bash
spec.source
```
* Defines the desired state, git repo url, branch and the path for argoCD to keep a watch on..

* This also, can be helm charts instead of git repo

```bash
spec.dest
```
Defines in which cluster and namespace, the manifest files suppose to be deployed

```bash
spec.syncPolicy
```
Defines, how the deployment should be managed. Should it be automated, prune resources, self-heal etc.