# GitOps Manifests - Argo CD Backend

This repository holds the **desired state** for the `argocd-backend` example. It contains only Kubernetes manifests — no application code and no CI workflows.

Argo CD watches `main` and the `k8s/` directory and keeps the cluster in sync with it.

## Repository layout

```text
GitOps/
├── .gitignore
├── README.md
└── k8s/
    ├── deployment.yaml
    └── service.yaml
```

## Manifests

`k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argocd-backend-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: argocd-backend
  template:
    metadata:
      labels:
        app: argocd-backend
    spec:
      containers:
        - name: argocd-backend
          image: abhinavchauhan24/argocd-backend:<short-sha>
          ports:
            - containerPort: 3000
```

`k8s/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: argocd-backend-service
spec:
  selector:
    app: argocd-backend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

## How this repo gets updated

The CI workflow in the application repository (`24thAbhinav/K8s`) builds the backend image, tags it with the commit's short SHA, and rewrites the `image:` line in `k8s/deployment.yaml` here. Nothing else is touched.

- Only `k8s/deployment.yaml` changes on a deploy.
- Replicas, Service settings, and labels stay under human control.
- The image is always tagged by SHA, never `latest`.

## Argo CD

The Application (`learning-argocd`) points at this repo:

```yaml
source:
  repoURL: https://github.com/24thAbhinav/GitOps.git
  targetRevision: main
  path: k8s
```

Auto-sync is on, with `selfHeal: true` and `prune: false`.

## Secrets

None here. Tokens (`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `GITOPS_TOKEN`) live as GitHub Actions secrets in the `K8s` repository.

## Full walkthrough

See [`basics/argocd/README.md`](https://github.com/24thAbhinav/K8s/blob/main/basics/argocd/README.md) in the application repository for the end-to-end setup, tokens, and troubleshooting.
