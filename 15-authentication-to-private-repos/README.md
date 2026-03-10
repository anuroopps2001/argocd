### Connecting to private repos via HTTPS
- we need to create a secret in `argocd` namespace providing the repo URL, username and password 
- also, we need to create a secret with label `argocd.argoproj.io/secret-type: repository` and this label helps argocd.


### For SSH connections to remote repos
- We have to create the secret with the help of private key and also, above mentioned label for secret is important
- Also, ssh keys are associated with speicific repository in github unlike PAT tokens which are associated with user accounts. Due to which using SSH keys is much better way than using PAT tokens and HTTPS authentication
