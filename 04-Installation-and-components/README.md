## Install argocd
```bash
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Argocd pods:
```bash
azureuser@azure-monitor-client-vm:~/argocd$ kubectl get pods -n argocd
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          67s
argocd-applicationset-controller-6799596c7c-nwr8v   1/1     Running   0          68s
argocd-dex-server-5cb8756cf7-w4k9g                  1/1     Running   0          68s
argocd-notifications-controller-5cd8948d4b-khnfz    1/1     Running   0          68s
argocd-redis-59784bcdb7-tb5l4                       1/1     Running   0          68s
argocd-repo-server-6868bb7494-hvgrm                 1/1     Running   0          68s
argocd-server-7488fb8dbf-km2tt                      1/1     Running   0          68s
azureuser@azure-monitor-client-vm:~/argocd$
```

To set argocd as default namespace in argocd:
```bash
kubectl config set-context --current --namespace=argocd
Context "minikube" modified.
```

## Components


### API Server
The API server is a gRPC/REST server which exposes the API consumed by the Web UI, CLI, and CI/CD systems. It has the following responsibilities:

- application management and status reporting
- invoking of application operations (e.g. sync, rollback, user-defined actions)
- repository and cluster credential management (stored as K8s secrets)
- authentication and auth delegation to external identity providers
- RBAC enforcement
- listener/forwarder for Git webhook events

### Repository Server

The repository server is an internal service which maintains a local cache of the Git repository holding the application manifests. 

It is responsible for generating and returning the Kubernetes manifests when provided the following inputs:

- repository URL
- revision (commit, tag, branch)
- application path
- template specific settings: parameters, helm values.yaml


### Application Controller
The application controller is a Kubernetes controller which continuously monitors running applications and compares the current, live state against the desired target state (as specified in the repo). It detects OutOfSync application state and optionally takes corrective action. 

It is responsible for invoking any user-defined hooks for lifecycle events (PreSync, Sync, PostSync)


## Application CRD in ArgoCD
The Application CRD is the Kubernetes resource object representing a deployed application instance in an environment. It is defined by two key pieces of information:

- `source` reference to the desired state in Git (repository, revision, path, environment)

- `destination` reference to the target cluster and namespace. For the cluster one of server or name can be used, but not both (which will result in an error). Under the hood when the server is missing, it is calculated based on the name and used for any operations.

Ex;
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
```

## AppProject CRD

The AppProject CRD is the Kubernetes resource object representing a logical grouping of applications. It is defined by the following key pieces of information:

- `sourceRepos` reference to the repositories that applications within the project can pull manifests from.

- `destinations` reference to clusters and namespaces that applications within the project can deploy into.

- `roles` list of entities with definitions of their access to resources within the project.


An example spec is as follows:
```bash
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
  # Finalizer that ensures that project is not deleted until it is not referenced by any application
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  description: Example Project
  # Allow manifests to deploy from any Git repos
  sourceRepos:
  - '*'
  # Only permit applications to deploy to the guestbook namespace in the same cluster
  destinations:
  - namespace: guestbook
    server: https://kubernetes.default.svc
  # Deny all cluster-scoped resources from being created, except for Namespace
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  # Allow all namespaced-scoped resources to be created, except for ResourceQuota, LimitRange, NetworkPolicy
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
  - group: ''
    kind: NetworkPolicy
  # Deny all namespaced-scoped resources from being created, except for Deployment and StatefulSet
  namespaceResourceWhitelist:
  - group: 'apps'
    kind: Deployment
  - group: 'apps'
    kind: StatefulSet
  roles:
  # A role which provides read-only access to all applications in the project
  - name: read-only
    description: Read-only privileges to my-project
    policies:
    - p, proj:my-project:read-only, applications, get, my-project/*, allow
    groups:
    - my-oidc-group
  # A role which provides sync privileges to only the guestbook-dev application, e.g. to provide
  # sync privileges to a CI system
  - name: ci-role
    description: Sync privileges for guestbook-dev
    policies:
    - p, proj:my-project:ci-role, applications, sync, my-project/guestbook-dev, allow
    # NOTE: JWT tokens can only be generated by the API server and the token is not persisted
    # anywhere by Argo CD. It can be prematurely revoked by removing the entry from this list.
    jwtTokens:
    - iat: 1535390316
```


