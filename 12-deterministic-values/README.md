### Kubernetes GitOps: Achieving Determinism with Argo CD.

#### The Problem with :latest 

Using the :latest tag (or any mutable tag like :dev or :stable) is considered an anti-pattern in production DevOps for several reasons.
- **Argo CD "Blindness"**: Argo CD compares the Desired State (Git) to the Actual State (Cluster). If Git says image: my-app:latest and the cluster is already running image: my-app:latest, Argo CD sees no difference, even if you just pushed new code to the registry.

- **Non-Deterministic Deployments**: Running the same "Sync" today might result in different code than running it tomorrow. 

- **No Rollback Capability**: You cannot easily "go back" to a previous version because the previous version was also named :latest and has been overwritten. 

- **Split-Brain Clusters**: If a node restarts, it might pull a newer version of :latest while other nodes in the same cluster continue running the older version.

#### Solution: The "Git SHA" Tagging Strategy
The industry standard is to tag every container image with the Git Commit Hash (SHA) from which it was built.

**The Workflow**:
- *Developer* pushes code to Git.
- *Jenkins* triggers a build and identifies the Commit SHA (e.g., a1b2c3d).
- *Docker* Build tags the image as my-registry.com/my-app:a1b2c3d.
- *Jenkins Update*: Jenkins uses a tool like yq or a shell script to update the values.yaml in your GitOps repository to point to the new tag.
- *Argo CD Sync*: Argo CD detects the change in the manifest (a1b2c3d vs the old SHA) and triggers a `Rolling Update`.

#### Using `fullnameOverride` for Deterministic Naming
In Helm, resource names are often generated dynamically by combining the Release Name and the Chart Name.

In your Argo CD Application YAML:
```bash
spec:
  source:
    helm:
      values: |
        fullnameOverride: "nginx-ing"
        controller:
          admissionWebhooks:
            enabled: true
```
Result: Your service will always be nginx-ing-controller, regardless of how long your Argo CD application name is.

#### Best Practices

```bash
COMMIT_SHA=$(git rev-parse --short HEAD)

# 2. Build and Push
docker build -t my-reg/my-app:${COMMIT_SHA} .
docker push my-reg/my-app:${COMMIT_SHA}

# 3. Update the GitOps Repo (using yq)
# This changes the tag in your values file so ArgoCD sees the "diff"
yq eval ".image.tag = \"${COMMIT_SHA}\"" -i ./charts/my-app/values.yaml

# 4. Commit and Push back to Git
git add .
git commit -m "chore: update image tag to ${COMMIT_SHA}"
git push origin main
```
