### Default rollout strategy in k8s
* In k8s, default rollout strategy is rollingUpdate and this will try to create the new version of pods as soon as possible without observing new versions behaviour and might lead to disruptions.

* Application running success is defined poorly 

* In RollingUpdate strategy, there is no concept of traffic control. As soon as new version pods comes and they will be added as backends for the service to serve the traffic without observing the application behaviour

* Rollback will happen only for limited revisions whick k8s had in memory
```bash
kubectl rollout history deployment/<deployment_name>

kubectl rollout undo deployment/<deployment_name>  # to roll back to previous version

kubectl rollout undo deployment/<deployment_name> --to-revision=<revision_number>  # rollback to specific previous version

kubectl rollout status deployment/<deployment_name>
```
