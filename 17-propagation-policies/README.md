**In K8s, the order of resource deletion is sequential**
i.e if we have deployment, and we delete the deployement, the deletion order would be
i. first pods will be deleted which created by replicaset
ii. replica set will be deleted which was created by deployment
iii. finally the deployment will be deleed


However, we can control the deletion order in argocd
### 1. FOREGROUND
Here, the owener(deployment) enters a `deletion in progress` state. The garbage collector first deletes all the dependencies(pods and replicasets) and at last deletes owner(deployment)

*Deletes the owner first, then children, but keeps the parent in a "deleting" state until children are gone.*

### 2. BACKGROUP
Here, the owner(deployment) object will be deleted immediately and garbage collector in the background deletes the dependent objects (pods and replicasets) in the background

*The Kubernetes API server deletes the owner immediately, while the controller deletes children in the background.*

- This is the default setting in ArgoCD

### 3. ORPHAN
Here, the owner will be deleted. However the dependent objects will be left behind. They become `orphaned`.
*Deletes the parent object but leaves dependent objects (like Pods) running in the cluster.*
