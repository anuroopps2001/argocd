## Helm Value Overrides in Argo CD

### 1. Using `valueFiles` (External Files)

This is used when you want to store your environment-specific settings (like production-values.yaml) inside your Git repository.
- Best for: Large configurations or keeping settings version-controlled in Git.
- How it works: You provide a list of paths to YAML files relative to the root of the chart.
```bash
spec:
  source:
    repoURL: 'https://github.com/my-org/my-infra.git'
    path: charts/my-app
    helm:
      valueFiles:
        - values.yaml           # Default chart values
        - values-production.yaml # Overrides specific to production
```


### 2. 2. Using `values` (Inline YAML String)

This is what we used to fix your `fullnameOverride`. It allows you to write YAML directly inside the Application CRD.

- Best for: Quick fixes, sensitive toggles, or small overrides that don't deserve their own file
- Important: You must use the pipe symbol (|) to treat the block as a multi-line string.
```yaml
spec:
  source:
    helm:
      values: |
        fullnameOverride: "nginx-ing"
        controller:
          replicaCount: 2
          service:
            type: LoadBalancer
```

### 3. Using `parameters` (Key-Value Pairs)
This is the simplest method. It allows you to override single values using a list of name/value pairs.

- Best for: Automated pipelines (like Jenkins) where you only need to update one specific field, such as an image tag.

- How it works: It behaves exactly like the --set flag in the Helm CLI.

```bash
spec:
  source:
    helm:
      parameters:
        - name: "controller.replicaCount"  # specific key
          value: "3"                       # related value
        - name: "image.tag"
          value: "v1.2.3"
```

### Order of Precedence (Priority)
If you define the same setting in multiple places, Argo CD follows a strict "Last One Wins" rule based on this hierarchy:

- parameters (Highest Priority - Always Wins)

- values (Inline String)

- valueFiles (The files listed lower in the list override those above them)

- Default values.yaml (Inside the Helm Chart - Lowest Priority)
