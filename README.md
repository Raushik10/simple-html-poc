# simple-html-poc

A tiny nginx page that proves the delivery pipeline: laptop (Docker Compose) -> staging -> production on minikube, built once and promoted by commit SHA.

## Flow

| Trigger | What happens |
| --- | --- |
| Pull request to `main` | Lint chart, build image, Trivy scan. Nothing is pushed or deployed |
| Push to `main` | Build, scan, push `ghcr.io/<owner>/<repo>/web:<sha>`, deploy to `staging`, run `helm test` |
| Tag `vX.Y.Z` | Verify the image for that commit exists (no rebuild), approval, deploy to `prod`, run `helm test` |

## One-time setup

1. Create a **private** GitHub repo and push these files to `main`.
2. Create a personal access token (classic) with only `read:packages`. Add it as repo secret `GHCR_PULL_TOKEN`.
3. Settings > Environments: create `staging` and `production`. Add required reviewers on `production` (private repos need a paid plan for this).
4. Start minikube and create the namespaces:

```bash
minikube start -p poc --driver=docker --cpus=2 --memory=4096
kubectl config use-context poc
kubectl create namespace staging
kubectl create namespace prod
```

5. Install a self-hosted runner (Settings > Actions > Runners) with the label `minikube`, and run it in a terminal where `kubectl get ns` works.

## Run locally

```bash
docker compose up --build
```

Open http://localhost:8080 (shows "Environment: local").

## Try the chart by hand

```bash
docker build -t web:dev --build-arg VERSION=manual app
minikube -p poc image load web:dev
helm upgrade --install web deploy/helm/web -n staging --set envName=manual
helm test web -n staging
kubectl -n staging port-forward svc/web 8081:80
helm uninstall web -n staging
```

## Deploy

```bash
git push origin main            # builds and deploys to staging
git tag v0.1.0 && git push origin v0.1.0   # promotes the same image to prod
kubectl -n prod port-forward svc/web 8082:80
```

## Rollback

```bash
helm history web -n prod
helm rollback web <revision> -n prod
```
