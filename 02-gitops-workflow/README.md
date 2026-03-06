### Gitops workflow

App Repository -> Pipeline that runs the several stages -> Update the k8s manifest in configuration repository ->
                                                                                    ask argocd to look for changes in cofig repository.


**Important point here is, keeping the separate repositories for application source code(App repo)and for the and k8s manifests repo(config repo)**