---
theme: default
title: Kubernetes - From Chaos to Orchestration
titleTemplate: "%s"
class: text-center
highlighter: shiki
lineNumbers: true
transition: slide-left
mdc: true
---

# Kubernetes
## From Monoliths to Orchestration

A practical introduction for software engineers

---
layout: section
---

# Part 1
## Why Does Kubernetes Exist?

---

## The Monolith Era

Most software started as one deployable application.

- One codebase, one deployment, one database
- Easy to build and ship early on
- Works well for small teams

```text
Monolith
├── Auth
├── Products
├── Orders
└── Database
```

---

## The Cracks Start to Show

As systems grow, monoliths become painful.

- Small changes require full redeploys
- Hard to scale only one hot path
- Bigger codebase slows feature work
- A bug in one area can impact everything

---

## Enter Microservices

Break one large app into smaller independent services.

- Service per domain (auth, orders, payments)
- Independent deploys per service
- Better team ownership boundaries
- Scale only what needs scaling

```text
API Gateway -> Auth / Product / Order / Payment Services
```

---

## New Powers, New Problems

Microservices solve one problem and create another.

- Many services to deploy and monitor
- More network communication between components
- Harder release coordination
- More operational complexity

---

## Containers to the Rescue (Briefly)

Docker made packaging and runtime consistency much easier.

- App + dependencies ship together
- Portable across environments
- Fast startup compared to full VMs

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Docker alone still does not orchestrate clusters.

---
layout: center
class: text-center
---

## The Orchestration Problem

At scale, containerized apps must:

- restart failed workloads
- scale up/down with demand
- route traffic correctly
- roll out updates safely
- handle config and secrets

**This is exactly what Kubernetes solves.**

---
layout: fact
---

# Kubernetes
## An open-source system for automating deployment, scaling, and management of containerized applications

Originally created at Google, now maintained by CNCF.

---
layout: section
---

# Part 2
## How Kubernetes Works

---

## The Big Picture: A Cluster

Kubernetes runs as a **cluster** of machines.

- Control Plane = decision making
- Worker Nodes = run containers
- You interact through `kubectl`

```text
You -> kubectl -> Control Plane -> Worker Nodes
```

---
layout: two-cols
---

## Pods - The Smallest Unit

A **Pod** is the smallest deployable unit.

- One or more containers per Pod
- Shared network + storage within the Pod
- Pod gets an internal cluster IP
- Pods are ephemeral

::right::

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: nginx:1.25
      ports:
        - containerPort: 80
```

---

## Deployments - Don't Run Pods Directly

Use Deployments to manage Pods safely.

- Maintain desired replica count
- Support rolling updates
- Enable rollbacks
- Replace unhealthy Pods

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
```

---

## ReplicaSets and Reconciliation

Deployments use ReplicaSets under the hood.

- Desired state: 3 Pods
- Actual state: 2 Pods
- Controller creates the missing Pod

Kubernetes continuously reconciles desired vs actual state.

---

## Services - Stable Networking

Pods are ephemeral; Service gives stable access.

- Stable DNS name and virtual IP
- Load balancing across healthy Pods
- Pod selection by labels

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

---

## Labels and Selectors - The Glue

Labels connect Kubernetes resources.

- Services find target Pods
- Deployments track managed Pods
- Operators filter resources precisely

```bash
kubectl get pods -l env=production
kubectl get pods -l app=payment-service,version=v3.2
```

---
layout: section
---

# Going Deeper
## Control Plane and Advanced Concepts

---

## The Control Plane

Core components:

- **API Server** - cluster entrypoint
- **etcd** - persistent cluster state
- **Scheduler** - assigns Pods to nodes
- **Controller Manager** - reconciliation loops

---

## Namespaces - Virtual Clusters

Namespaces partition one physical cluster.

- Separate `dev`, `staging`, `production`
- Isolate teams and projects
- Avoid naming collisions

```bash
kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
kubectl get pods --all-namespaces
```

---
layout: two-cols
---

## ConfigMaps and Secrets

Keep configuration outside container images.

- ConfigMap for non-sensitive values
- Secret for sensitive data
- Easier environment-specific configuration

::right::

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: info
  FEATURE_X: "true"
```

---

## Ingress - External Traffic

Ingress controls inbound traffic into the cluster.

- Host/path routing
- TLS termination
- Clean exposure of internal services

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

---

## Persistent Volumes

Containers are temporary; data often is not.

- **PV** = storage resource
- **PVC** = storage request
- Stateful workloads mount claims for durability

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 20Gi
```

---

## Health Checks - Liveness and Readiness

Probes help route traffic only to healthy Pods.

- Liveness: should container be restarted?
- Readiness: can it receive traffic now?

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 3000
readinessProbe:
  httpGet:
    path: /ready
    port: 3000
```

---

## Resource Requests and Limits

Define resource intent per container.

- Requests reserve CPU/memory
- Limits cap maximum consumption
- Improves scheduling and multi-tenant fairness

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

---

## Helm - Package Manager for Kubernetes

Helm packages Kubernetes manifests as reusable charts.

- Chart = templated app package
- Values = environment customization
- Release upgrades and rollbacks

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql --set primary.persistence.size=20Gi
helm upgrade my-postgres bitnami/postgresql --set image.tag=16.2
helm rollback my-postgres 1
```

---
layout: center
class: text-center
---

## The Kubernetes Ecosystem

Kubernetes is the platform core with a large ecosystem:

- Managed K8s: EKS, GKE, AKS
- Observability: Prometheus, Grafana, Jaeger
- Delivery: Argo CD, Flux, Tekton
- Service Mesh: Istio, Linkerd

---
layout: two-cols
---

## Putting It All Together

A typical request flow:

```text
Browser
  -> Load Balancer
  -> Ingress
  -> Service
  -> Pod (application)
  -> Service
  -> Pod (database) + PersistentVolume
```

::right::

### What Kubernetes gives you

- Self-healing
- Scaling
- Rolling deploys
- Service discovery
- Load balancing
- Config + secret management
- Resource isolation
- Portability

---
layout: center
---

## Key `kubectl` Commands

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl describe pod my-pod-xyz
kubectl logs -f my-pod-xyz
kubectl exec -it my-pod-xyz -- /bin/sh
kubectl apply -f deployment.yaml
kubectl scale deployment my-app --replicas=5
kubectl top nodes
kubectl top pods
```

---
layout: end
class: text-center
---

# Summary

## The Journey

- Monoliths -> Microservices -> Containers
- Containers created orchestration complexity
- Kubernetes solves it with declarative operations

## Core Concepts

- Cluster -> Nodes -> Pods
- Deployments manage replicas and updates
- Services provide stable networking
- Ingress handles external routing
- ConfigMaps, Secrets, and PVCs manage runtime needs

**Next step:** run a local cluster with [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io)
