# Kubernetes: SynergyChat

Kubernetes manifests deploying a multi-service chat application (web frontend, API and a keyword crawler), written while completing Boot.dev's Learn Kubernetes course. The container images are provided by the course; the manifests are mine.

## Architecture

```
            Gateway (Envoy Gateway, port 80)
             │                         │
   synchat.internal          synchatapi.internal
             │                         │
       web-service               api-service
             │                         │
   synergychat-web           synergychat-api ──► crawler-service (namespace: crawler)
   (HPA: 1 to 4 pods)        (PVC-backed JSON DB)        │
                                                synergychat-crawler
                                                (3 containers, shared emptyDir cache)
```

## What it covers

| Concept | Where |
|---|---|
| Deployments and Services | `*-deployment.yaml`, `*-service.yaml` |
| Config injected from ConfigMaps (`envFrom` and per-key `configMapKeyRef`) | `*-configmap.yaml` |
| Persistent storage with a PersistentVolumeClaim | `api-pvc.yaml`, mounted at `/persist` by the API |
| Shared ephemeral storage between containers in one pod | crawler's `emptyDir` volume at `/cache` |
| Cross-namespace service discovery | API reaches `crawler-service.crawler.svc.cluster.local` |
| Ingress via the Gateway API (GatewayClass, Gateway, HTTPRoutes) | `app-gateway*.yaml`, `*-httproute.yaml` |
| Horizontal Pod Autoscaling on CPU | `web-hpa.yaml`, `testcpu-hpa.yaml` |
| Resource limits and requests | `testcpu-deployment.yaml`, `testram-development.yaml` |

The `testcpu` and `testram` deployments are deliberate stress cases. `testcpu` runs against a tight CPU limit to exercise the autoscaler. `testram` requests more memory than a local cluster has, to show a pod stuck in `Pending` when the scheduler can't place it.

## Running it

Requires a local cluster (for example minikube), `kubectl`, and [Envoy Gateway](https://gateway.envoyproxy.io/) installed as the Gateway API controller.

```bash
kubectl create namespace crawler
kubectl apply -f .
```

Map `synchat.internal` and `synchatapi.internal` to the gateway's address in `/etc/hosts` to reach the app.
