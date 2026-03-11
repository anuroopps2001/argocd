### Rollout CRD
* Rollout yaml files are almost similar to Deployment yaml files. However,
```bash
apiVersion: argoproj.io/v1alpha1
```

and
```bash
kind: Rollout
```

* Rollout's support Blue-green and canary rollout strategies unlinke deployments which supports only Rolling Updates

### Deploying rollouts 
```yaml
ubuntu@ip-172-31-3-126:~/argocd/22-first-rollout-crd$ kubectl-argo-rollouts list rollouts
NAME              STRATEGY   STATUS        STEP  SET-WEIGHT  READY  DESIRED  UP-TO-DATE  AVAILABLE
simple-color-app  unknown    Degraded      -     -           0/0    5        0           0
ubuntu@ip-172-31-3-126:~/argocd/22-first-rollout-crd$ kubectl-argo-rollouts get rollout simple-color-app
Name:            simple-color-app
Namespace:       default
Status:          ✖ Degraded
Message:         InvalidSpec: The Rollout "simple-color-app" is invalid: spec.strategy.strategy: Required value: Rollout has missing field '.spec.strategy.canary or .spec.strategy.blueGreen'
Strategy:
Replicas:
  Desired:       5
  Current:       0
  Updated:       0
  Ready:         0
  Available:     0

NAME                KIND     STATUS      AGE  INFO
⟳ simple-color-app  Rollout  ✖ Degraded  53s
```

**Deploying rollouts without any strategies results in rollout degradation**
