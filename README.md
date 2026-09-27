# AI-Driven Kubernetes Operations

Six ASP.NET Core APIs deployed to a local Kubernetes cluster through GitOps, with production-grade manifests and AI-assisted operations tooling.

## How It Works

```
docker compose build  ->  local images  ->  ArgoCD watches k8s/  ->  cluster
```

ArgoCD syncs each service from `k8s/<service>/` on the default branch with auto-prune and self-heal, so the cluster always matches Git.

## Layout

```
App.API ... App6.API   six independent ASP.NET Core APIs
k8s/<service>/         namespace, configmap, secret, deployment, service, hpa
docker-compose.yml     builds all service images locally
```

Each service gets its own namespace (`app2-system`, `app5-system`, ...) and exposes `/weatherforecast` plus health endpoints.

## Manifest Highlights

- 2 replicas, zero-downtime rolling updates (`maxUnavailable: 0`)
- HPA: 2-10 replicas on 70% CPU / 80% memory
- `readOnlyRootFilesystem`, `runAsNonRoot`, all Linux capabilities dropped, `seccompProfile: RuntimeDefault`
- Topology spread constraints across nodes
- `GET /healthz/live` for liveness, `GET /healthz/ready` for readiness

## Getting Started

```bash
git clone https://github.com/Fcakiroglu16/ai-driven-kubernetes-operations.git
cd ai-driven-kubernetes-operations

docker compose build

argocd app create app2-api \
  --repo https://github.com/Fcakiroglu16/ai-driven-kubernetes-operations.git \
  --path k8s/app2-api \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace app2-system \
  --sync-policy automated --auto-prune --self-heal

kubectl get pods -n app2-system
```

Images use `imagePullPolicy: IfNotPresent`, so no registry push is needed.

## Requirements

- .NET SDK, Docker Desktop
- kubectl and a local cluster (kind or docker-desktop)
- ArgoCD installed in the `argocd` namespace
