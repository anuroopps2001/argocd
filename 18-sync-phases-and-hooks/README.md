### Sync phases and hooks

#### 1. presync
when we trigger a sync, argocd does 
- **presync** i.e execute the jobs which having `presync` hook under annotations


#### 2. sync
Once the `presync` hook jobs are executed, argocd enters into `sync` phase and execute the jobs if any having annotation `argocd.argoproj.io/hook: Sync` and also sync the manifests files.

- If the sync is successful, argocd enters into `postsync` phase else it will go inot `syncFail` phase and execute the jobs if any with annotation `argocd.argoproj.io/hook: SyncFail`

#### 3. syncFail

- If sync fails, execute the jobs with `SyncFail` hook

#### 4. PostSync
If the sync process completes successfully, then argocd will execute the jobs if any with annotations `argocd.argoproj.io/hook: PostSync`
