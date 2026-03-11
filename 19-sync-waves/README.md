### Sync Waves

* Normally resources in specific sync phase, that might be in pre-sync, sync or in post-sync will be created roughly at the same time in respective sync phase

* However, if we want to maintain an execution order for resources in specific sync phase, we make use `sync waves`.

#### Defining sync waves
- Add a annotation `argocd.argoproj.io/sync-wave: ` with a -ve, 0 or +ve numbers 

Ex:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mydb
  annotations:
    argocd.argoproj.io/sync-wave: "2"  # this number can be +ve integer, -ve interger and default value is 0
```

**The order of execution of deployment of each resource depends on it;s sync-wave value within an specific sync phase starting from -ve value to +ve value resources**


