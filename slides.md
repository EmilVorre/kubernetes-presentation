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
class: text-center
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
---
theme: default
title: Kubernetes - From Chaos to Orchestration
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
## Why Kubernetes Exists

---

## The Monolith Era

- One codebase, one database, one deployment
- Great for small teams and simple products
- Fast to ship initially

```text
Monolith -> App + Database
```

---

## The Cracks Start to Show

- Full redeploy for small changes
- Hard to scale only one hot component
- Large codebase slows teams
- One failure can impact everything

---

## Enter Microservices

- Split by domain: auth, orders, payments
- Deploy services independently
- Scale each service by demand
- Better team ownership boundaries

---

## New Powers, New Problems

- Many services to run and observe
- Service-to-service networking complexity
- More versions and environments to manage

---

## Containers to the Rescue

- Package app + dependencies together
- Consistent runtime across environments
- Lightweight and fast to start

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

---
layout: center
class: text-center
---

## The Orchestration Problem

At scale, containers must:

- restart on failure
- scale with load
- discover each other
- roll out updates safely
- manage config and secrets

**Kubernetes solves this.**

---
layout: fact
---

# Kubernetes
## Automates deployment, scaling, and management of containerized applications

---
layout: section
---

# Part 2
## How Kubernetes Works

---

## Cluster Basics

- Control Plane = brain
- Worker Nodes = run workloads
- You interact with the cluster via `kubectl`

```text
You -> kubectl -> Control Plane -> Nodes
```

---
layout: two-cols
---

## Pods - The Smallest Unit

- One or more containers per Pod
- Shared network/storage inside the Pod
- Pod IP inside cluster network
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

## Deployments

- Manage desired replica count
- Roll out updates gradually
- Roll back when needed
- Self-heal by replacing failed Pods

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
```

---

## Services and Labels

- Services provide stable DNS/IP for Pods
- Load balancing across healthy Pods
- Labels/selectors connect Services to Pods

```bash
kubectl get pods -l env=production
kubectl get pods -l app=payment-service,version=v3.2
```

---
layout: section
---

# Going Deeper
## Core Platform Features

---

## Control Plane Components

- API Server: front door
- etcd: source of truth
- Scheduler: places Pods
- Controllers: reconcile desired state

---

## Namespaces

- Separate environments (`dev`, `staging`, `prod`)
- Isolate teams/projects
- Avoid naming collisions

```bash
kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
kubectl get pods --all-namespaces
```

---

## Config and Secrets

- ConfigMap for non-sensitive config
- Secret for credentials/tokens
- Keep config outside container images

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: info
```

---

## Ingress and Persistent Storage

- Ingress routes external traffic by host/path
- PVCs provide persistent storage for stateful apps
- Essential for databases and durable data

```yaml
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

## Probes and Resource Limits

- Liveness/readiness probes improve reliability
- Requests/limits improve scheduling fairness

```yaml
resources:
  requests: { cpu: "250m", memory: "256Mi" }
  limits: { cpu: "500m", memory: "512Mi" }
```

---

## Helm

- Chart = packaged Kubernetes templates
- Values = environment-specific customization
- Easy upgrades and rollbacks

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql --set primary.persistence.size=20Gi
helm rollback my-postgres 1
```

---
layout: two-cols
---

## End-to-End Request Flow

```text
Browser -> Load Balancer -> Ingress -> Service -> Pod
```

```text
App Pod -> Service -> Database Pod + PersistentVolume
```

::right::

### What Kubernetes gives you

- Self-healing
- Scaling
- Rolling deploys
- Service discovery
- Config/secret management
- Portability

---

## Key `kubectl` Commands

```bash
kubectl get pods
kubectl get deployments
kubectl describe pod my-pod
kubectl logs -f my-pod
kubectl exec -it my-pod -- /bin/sh
kubectl apply -f deployment.yaml
kubectl scale deployment my-app --replicas=5
kubectl top nodes
```

---
layout: end
class: text-center
---

# Summary

- Monoliths -> Microservices -> Containers
- Containers created orchestration complexity
- Kubernetes provides declarative operations at scale
- Core objects: Pods, Deployments, Services, Ingress, Config, Storage

**Next step:** try [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io)
---
theme: default
title: Kubernetes - From Chaos to Orchestration
titleTemplate: "%s"
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
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
layout: two-cols
---

## The Monolith Era

In the beginning, most software was a **single deployable unit**.

- One codebase, one database, one deployment
- All features in one process
- Great for small teams and early products
- Simple to understand and ship

::right::

```text
Monolith
├── Auth
├── Products
├── Orders
├── Payments
└── One Database
```

---

## The Cracks Start to Show

As applications grow, monoliths become painful.

- One small change can force a full redeploy
- Scaling means scaling everything, not just hot paths
- Bigger codebases slow teams down
- Failures in one part can impact the whole app

---
layout: two-cols
---

## Enter Microservices

Split a large app into smaller, independent services.

- Each service owns one domain (auth, orders, payments)
- Services can be deployed independently
- Teams move faster with smaller codebases
- Services can scale based on actual demand

::right::

```text
API Gateway
├── Auth Service
├── Product Service
├── Order Service
└── Payment Service
```

---

## New Powers, New Problems

Microservices solve software design problems but create operational complexity.

- Many services to deploy and monitor
- Many network connections between services
- Multiple versions running at once
- More moving pieces across environments

---

## Containers to the Rescue (Briefly)

Docker made packaging and shipping applications consistent.

- Containers bundle app + dependencies
- "Works on my machine" becomes portable
- Faster startup than full virtual machines
- Built from a repeatable `Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Docker alone still doesn't orchestrate hundreds of containers.

---
layout: center
class: text-center
---

## The Orchestration Problem

You have many containers. They need to:

- Restart when they crash
- Scale up and down with traffic
- Discover each other across machines
- Deploy with minimal downtime
- Spread across infrastructure safely
- Handle config, secrets, and networking

**This is the gap Kubernetes fills.**

---
layout: fact
---

# Kubernetes
## An open-source system for automating deployment, scaling, and management of containerized applications

Originally from Google, now maintained by CNCF.

---
layout: section
---

# Part 2
## How Kubernetes Works

Starting from the ground up.

---

## The Big Picture: A Cluster

Kubernetes runs as a **cluster** of machines.

- You interact with the cluster, not individual servers
- Control Plane = decision making
- Worker Nodes = run workloads

```text
You -> kubectl -> Control Plane -> Worker Nodes
```

---
layout: two-cols
---

## Pods - The Smallest Unit

A **Pod** is Kubernetes' smallest deployable unit.

- Usually runs one main container
- Pod containers share networking and storage
- Each Pod gets an internal IP
- Pods are ephemeral and replaceable

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

Use **Deployments** to manage Pods safely.

- Keep desired number of replicas
- Handle rolling updates
- Enable rollbacks
- Recreate failed Pods automatically

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
```

---
layout: two-cols
---

## ReplicaSets and Reconciliation

The Deployment uses a **ReplicaSet** under the hood.

- Desired state: "I want 3 Pods"
- Actual state: "Only 2 are running"
- Kubernetes reconciles until desired = actual

::right::

```text
Desired: 3 replicas
Actual: 2 replicas
Controller: create 1 new Pod
```

---

## Services - Stable Networking

Pods are ephemeral; their IPs can change. **Services** provide stability.

- Stable DNS name for a Pod group
- Built-in load balancing
- Select target Pods by labels

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

Labels connect Kubernetes objects.

- Services select Pods using labels
- Deployments track managed Pods by labels
- Operators query and group resources with labels

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
layout: two-cols
---

## The Control Plane

The control plane is the cluster brain.

- **API Server**: entrypoint for all operations
- **etcd**: persistent cluster state
- **Scheduler**: assigns Pods to nodes
- **Controller Manager**: reconciliation loops

::right::

```text
You -> API Server -> etcd
                 -> Scheduler
                 -> Controllers
```

---

## Namespaces - Virtual Clusters

Namespaces partition one cluster into isolated environments.

- Separate `dev`, `staging`, `production`
- Organize by team or project
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

Keep config outside images.

- **ConfigMap**: non-sensitive settings
- **Secret**: sensitive values (tokens, passwords)
- Improves portability across environments

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

Ingress exposes services to the outside world.

- Route by host/path
- Centralize TLS termination
- Keep service networking internal

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

Containers are ephemeral. Data usually isn't.

- **PersistentVolume (PV)** = storage resource
- **PersistentVolumeClaim (PVC)** = requested storage
- Pods mount claims for durable state

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

## Health Checks

Kubernetes uses probes to know container health.

- **Liveness probe**: should this container be restarted?
- **Readiness probe**: is this container ready for traffic?
- Prevents sending users to unhealthy Pods

```yaml
livenessProbe:
  httpGet: { path: /healthz, port: 3000 }
readinessProbe:
  httpGet: { path: /ready, port: 3000 }
```

---

## Resource Requests and Limits

Define resource intent so scheduling is predictable.

- **Requests** reserve CPU/memory
- **Limits** cap maximum usage
- Prevent noisy-neighbor issues
- Improve cluster utilization

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

Real apps require many manifests. Helm manages that complexity.

- **Chart** = packaged templates
- **Values** = environment customization
- Versioned releases + rollbacks

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql --set primary.persistence.size=20Gi
helm upgrade my-postgres bitnami/postgresql --set image.tag=16.2
helm rollback my-postgres 1
```

Think of Helm like npm/pip for Kubernetes manifests.

---
layout: center
class: text-center
---

## The Kubernetes Ecosystem

Kubernetes is the platform core; tools build on top.

- Managed services: EKS, GKE, AKS
- Observability: Prometheus, Grafana, Jaeger
- GitOps/CI-CD: Argo CD, Flux, Tekton
- Service mesh: Istio, Linkerd

---
layout: two-cols
---

## Putting It All Together

A typical request path:

```text
Browser
  -> Load Balancer
  -> Ingress
  -> Service
  -> Pod (app)
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
class: text-center
---

## Key `kubectl` Commands

```bash
kubectl get pods
kubectl get deployments
kubectl get services

kubectl describe pod my-pod-xyz

kubectl logs my-pod-xyz
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
- Containers introduced orchestration complexity
- Kubernetes solves it with declarative operations

## Core Concepts

- Cluster -> Nodes -> Pods
- Deployments manage replicas and rollouts
- Services provide stable networking
- Ingress handles external routing
- ConfigMaps and Secrets manage configuration

**Next step:** run a local cluster with [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io)
---
theme: default
title: Kubernetes - From Chaos to Orchestration
titleTemplate: "%s"
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
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
layout: two-cols
---

## The Monolith Era

In the beginning, most software was a **single deployable unit**.

- One codebase, one database, one deployment
- All features in one process
- Great for small teams and early products
- Simple to understand and ship

::right::

```text
Monolith
├── Auth
├── Products
├── Orders
├── Payments
└── One Database
```

---

## The Cracks Start to Show

As applications grow, monoliths become painful.

- One small change can force a full redeploy
- Scaling means scaling everything, not just hot paths
- Bigger codebases slow teams down
- Failures in one part can impact the whole app

---
layout: two-cols
---

## Enter Microservices

Split a large app into smaller, independent services.

- Each service owns one domain (auth, orders, payments)
- Services can be deployed independently
- Teams move faster with smaller codebases
- Services can scale based on actual demand

::right::

```text
API Gateway
├── Auth Service
├── Product Service
├── Order Service
└── Payment Service
```

---

## New Powers, New Problems

Microservices solve software design problems but create operational complexity.

- Many services to deploy and monitor
- Many network connections between services
- Multiple versions running at once
- More moving pieces across environments

---

## Containers to the Rescue (Briefly)

Docker made packaging and shipping applications consistent.

- Containers bundle app + dependencies
- "Works on my machine" becomes portable
- Faster startup than full virtual machines
- Built from a repeatable `Dockerfile`

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Docker alone still doesn't orchestrate hundreds of containers.

---
layout: center
class: text-center
---

## The Orchestration Problem

You have many containers. They need to:

- Restart when they crash
- Scale up and down with traffic
- Discover each other across machines
- Deploy with minimal downtime
- Spread across infrastructure safely
- Handle config, secrets, and networking

**This is the gap Kubernetes fills.**

---
layout: fact
---

# Kubernetes
## An open-source system for automating deployment, scaling, and management of containerized applications

Originally from Google, now maintained by CNCF.

---
layout: section
---

# Part 2
## How Kubernetes Works

Starting from the ground up.

---

## The Big Picture: A Cluster

Kubernetes runs as a **cluster** of machines.

- You interact with the cluster, not individual servers
- Control Plane = decision making
- Worker Nodes = run workloads

```text
You -> kubectl -> Control Plane -> Worker Nodes
```

---
layout: two-cols
---

## Pods - The Smallest Unit

A **Pod** is Kubernetes' smallest deployable unit.

- Usually runs one main container
- Pod containers share networking and storage
- Each Pod gets an internal IP
- Pods are ephemeral and replaceable

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

Use **Deployments** to manage Pods safely.

- Keep desired number of replicas
- Handle rolling updates
- Enable rollbacks
- Recreate failed Pods automatically

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app
```

---
layout: two-cols
---

## ReplicaSets and Reconciliation

The Deployment uses a **ReplicaSet** under the hood.

- Desired state: "I want 3 Pods"
- Actual state: "Only 2 are running"
- Kubernetes reconciles until desired = actual

::right::

```text
Desired: 3 replicas
Actual: 2 replicas
Controller: create 1 new Pod
```

---

## Services - Stable Networking

Pods are ephemeral; their IPs can change. **Services** provide stability.

- Stable DNS name for a Pod group
- Built-in load balancing
- Select target Pods by labels

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

Labels connect Kubernetes objects.

- Services select Pods using labels
- Deployments track managed Pods by labels
- Operators query and group resources with labels

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
layout: two-cols
---

## The Control Plane

The control plane is the cluster brain.

- **API Server**: entrypoint for all operations
- **etcd**: persistent cluster state
- **Scheduler**: assigns Pods to nodes
- **Controller Manager**: reconciliation loops

::right::

```text
You -> API Server -> etcd
                 -> Scheduler
                 -> Controllers
```

---

## Namespaces - Virtual Clusters

Namespaces partition one cluster into isolated environments.

- Separate `dev`, `staging`, `production`
- Organize by team or project
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

Keep config outside images.

- **ConfigMap**: non-sensitive settings
- **Secret**: sensitive values (tokens, passwords)
- Improves portability across environments

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

Ingress exposes services to the outside world.

- Route by host/path
- Centralize TLS termination
- Keep service networking internal

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

Containers are ephemeral. Data usually isn't.

- **PersistentVolume (PV)** = storage resource
- **PersistentVolumeClaim (PVC)** = requested storage
- Pods mount claims for durable state

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

## Health Checks

Kubernetes uses probes to know container health.

- **Liveness probe**: should this container be restarted?
- **Readiness probe**: is this container ready for traffic?
- Prevents sending users to unhealthy Pods

```yaml
livenessProbe:
  httpGet: { path: /healthz, port: 3000 }
readinessProbe:
  httpGet: { path: /ready, port: 3000 }
```

---

## Resource Requests and Limits

Define resource intent so scheduling is predictable.

- **Requests** reserve CPU/memory
- **Limits** cap maximum usage
- Prevent noisy-neighbor issues
- Improve cluster utilization

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

Real apps require many manifests. Helm manages that complexity.

- **Chart** = packaged templates
- **Values** = environment customization
- Versioned releases + rollbacks

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-postgres bitnami/postgresql --set primary.persistence.size=20Gi
helm upgrade my-postgres bitnami/postgresql --set image.tag=16.2
helm rollback my-postgres 1
```

Think of Helm like npm/pip for Kubernetes manifests.

---
layout: center
class: text-center
---

## The Kubernetes Ecosystem

Kubernetes is the platform core; tools build on top.

- Managed services: EKS, GKE, AKS
- Observability: Prometheus, Grafana, Jaeger
- GitOps/CI-CD: Argo CD, Flux, Tekton
- Service mesh: Istio, Linkerd

---
layout: two-cols
---

## Putting It All Together

A typical request path:

```text
Browser
  -> Load Balancer
  -> Ingress
  -> Service
  -> Pod (app)
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
class: text-center
---

## Key `kubectl` Commands

```bash
kubectl get pods
kubectl get deployments
kubectl get services

kubectl describe pod my-pod-xyz

kubectl logs my-pod-xyz
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
- Containers introduced orchestration complexity
- Kubernetes solves it with declarative operations

## Core Concepts

- Cluster -> Nodes -> Pods
- Deployments manage replicas and rollouts
- Services provide stable networking
- Ingress handles external routing
- ConfigMaps and Secrets manage configuration

**Next step:** run a local cluster with [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io)

---
theme: seriph
title: Kubernetes — From Chaos to Orchestration
titleTemplate: '%s'
background: https://images.unsplash.com/photo-1667372393119-3d4c48d07fc9?w=1920
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
transition: slide-left
mdc: true
---
 
# Kubernetes
## From Monoliths to Orchestration
 
A practical introduction for software engineers
 
<div class="pt-12 text-gray-400">
  Press <kbd>Space</kbd> to continue
</div>

---
layout: section
---
 
# Part 1
## Why Does Kubernetes Exist?
 
<!--
Start here by asking: who has built a web app before? Who has deployed one? This sets the stage for the problem.
-->
 
---
layout: two-cols
---
 
## The Monolith Era
 
In the beginning, most software was built as a **single deployable unit** — the monolith.
 
<v-clicks>
- One codebase, one database, one deployment
- All features live in the same process
- Works great when the team and app are small
- Familiar examples: early Facebook, Twitter, Shopify
</v-clicks>
::right::
 
<div class="pl-8 pt-4">
```
┌─────────────────────────────┐
│        MONOLITH APP         │
│                             │
│  ┌──────────┐ ┌──────────┐  │
│  │  Auth    │ │ Products │  │
│  └──────────┘ └──────────┘  │
│  ┌──────────┐ ┌──────────┐  │
│  │ Orders   │ │ Payments │  │
│  └──────────┘ └──────────┘  │
│  ┌──────────┐ ┌──────────┐  │
│  │  Email   │ │  Search  │  │
│  └──────────┘ └──────────┘  │
│                             │
│         └──────┘            │
│          ONE DB             │
└─────────────────────────────┘
```
 
</div>
<!--
Emphasize that monoliths aren't inherently bad — they're often the right choice early on. The problems come with growth.
-->
 
---
 
## The Cracks Start to Show
 
As applications grow, monoliths become painful to work with.
 
<v-clicks>
- 🐌 **Slow deployments** — changing one line means re-deploying everything
- 😱 **Scaling problems** — you can only scale *the entire app*, not just the busy part
- 🔒 **Technology lock-in** — the whole app must use the same language and framework
- 💥 **Fragile deploys** — one bad module can crash the entire service
- 👥 **Team bottlenecks** — 50 engineers fighting over one repository
</v-clicks>
<v-click>
> "Deploying our monolith was like open-heart surgery — we needed the entire team, a maintenance window, and a prayer."
 
</v-click>
<!--
Ask the audience: imagine 200 engineers all committing to the same repo daily. Merge conflicts, broken builds, coordinating releases — a nightmare.
-->
 
---
layout: image-right
image: https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=800
---
 
## Enter Microservices
 
**The idea:** break the monolith into small, independent services — each responsible for one thing.
 
<v-clicks>
- Each service has its **own codebase** and **its own database**
- Services communicate over the network (HTTP/gRPC)
- Teams can **deploy independently**
- You can **scale only what needs scaling**
- Mix and match languages and frameworks
</v-clicks>
<v-click>
This was the architecture that allowed Netflix, Uber, and Amazon to scale to millions of users.
 
</v-click>
<!--
Amazon famously has thousands of microservices. Jeff Bezos mandated in 2002 that all teams must expose their data via service APIs.
-->
 
---
layout: two-cols
---
 
## New Powers, New Problems
 
Microservices gave teams **velocity** — but introduced a whole new category of operational pain.
 
<v-clicks>
- You now have **50+ services** to deploy and monitor
- Each service needs its own server/VM — **expensive and slow to provision**
- How do services **find each other** on the network?
- What happens when **a service crashes**? Who restarts it?
- How do you **roll out updates** without downtime?
- How do you manage **configuration and secrets** across all services?
</v-clicks>
::right::
 
<div class="pl-8 pt-8">
```
user-service      → 3 instances
product-service   → 5 instances
order-service     → 2 instances
payment-service   → 4 instances
email-service     → 1 instance
search-service    → 8 instances
review-service    → 2 instances
auth-service      → 6 instances
notification-svc  → 1 instance
analytics-service → 3 instances
...
```
 
<div class="mt-4 text-orange-400 text-sm">
😰 Who manages all of this?
</div>
</div>
<!--
This is the key insight: the shift from monolith to microservices solved a software problem but created an infrastructure problem.
-->
 
---
 
## Containers to the Rescue (Briefly)
 
Before Kubernetes, Docker changed how we **package and run** applications.
 
<v-clicks>
- A **container** bundles your app with all its dependencies into a single portable unit
- Runs consistently on any machine: *"works on my machine"* → actually works everywhere
- Much lighter than a full virtual machine — starts in milliseconds
- Defined with a simple `Dockerfile`
</v-clicks>
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```
<v-click>
**But Docker alone just runs containers on one machine.** Who runs them across hundreds of servers?
 
</v-click>
<!--
Docker (2013) was a massive shift. But the question quickly became: great, I have 200 containers — now what?
-->
 
---
layout: center
class: text-center
---
 
## The Orchestration Problem
 
<div class="text-xl text-gray-400 mb-8">You have 200 containers. They need to:</div>
<div class="grid grid-cols-3 gap-6 text-left">
<v-clicks>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">🔄</div>
<strong>Auto-restart</strong> when they crash
</div>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">📈</div>
<strong>Scale up/down</strong> based on traffic
</div>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">🔍</div>
<strong>Find each other</strong> across servers
</div>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">🚀</div>
<strong>Deploy updates</strong> with zero downtime
</div>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">⚖️</div>
<strong>Distribute load</strong> across machines
</div>
<div class="bg-blue-900/50 rounded-lg p-4">
<div class="text-2xl mb-2">🔐</div>
<strong>Manage secrets</strong> and config safely
</div>
</v-clicks>
</div>
<v-click>
<div class="mt-8 text-2xl font-bold text-blue-400">This is exactly what Kubernetes does.</div>
</v-click>
<!--
This slide is the punchline of Part 1. Everything before this was building to this moment. Kubernetes solves all of these problems.
-->
 
---
layout: fact
---
 
# Kubernetes
## An open-source system for **automating deployment, scaling, and management** of containerized applications
 
<div class="text-gray-400 mt-4">Originally designed by Google. Open-sourced in 2014. Now maintained by the CNCF.</div>
<!--
Google had been running containers internally with a system called Borg for over a decade. Kubernetes is the open-source evolution of those learnings.
-->
 
---
layout: section
---
 
# Part 2
## How Kubernetes Works
 
*Starting from the ground up*
 
---
 
## The Big Picture: A Cluster
 
Kubernetes runs as a **cluster** — a group of machines that work together as one system.
 
<v-clicks>
- You don't talk to individual machines — you talk to **the cluster**
- The cluster has two types of nodes:
  - **Control Plane** (the brain) — makes decisions
  - **Worker Nodes** (the muscle) — actually run your containers
</v-clicks>
```
┌──────────────────────────────────────────────────────┐
│                   KUBERNETES CLUSTER                  │
│                                                      │
│  ┌──────────────────┐   ┌───────┐ ┌───────┐ ┌─────┐ │
│  │  CONTROL PLANE   │   │ Node  │ │ Node  │ │Node │ │
│  │  (the brain)     │   │  1    │ │  2    │ │  3  │ │
│  └──────────────────┘   └───────┘ └───────┘ └─────┘ │
│                                                      │
│         You → kubectl → Control Plane → Nodes        │
└──────────────────────────────────────────────────────┘
```
<!--
A key mental model: you never tell Kubernetes WHERE to run something. You tell it WHAT you want, and it figures out the where.
-->
 
---
layout: two-cols
---
 
## Pods — The Smallest Unit
 
A **Pod** is the smallest deployable unit in Kubernetes.
 
<v-clicks>
- A Pod wraps **one or more containers**
- Containers in a Pod share network and storage
- Every Pod gets its **own IP address** inside the cluster
- Pods are **ephemeral** — they can die and be replaced at any time
</v-clicks>
<v-click>
Think of a Pod like a tiny virtual machine dedicated to one task.
 
</v-click>
::right::
 
<div class="pl-8 pt-4">
```yaml
# The simplest Pod definition
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
 
<div class="mt-4 text-sm text-gray-400">
This is YAML — the language of Kubernetes config
</div>
</div>
<!--
The YAML here is intentionally simple. Don't dwell on syntax — the key concepts are: Pod wraps containers, ephemeral, own IP.
-->
 
---
 
## Deployments — Don't Run Pods Directly
 
In practice, you almost never create Pods manually. You create a **Deployment**.
 
<v-clicks>
- A Deployment says: *"I want 3 copies of this Pod running at all times"*
- Kubernetes ensures that number is **always maintained**
- If a Pod crashes → Kubernetes **automatically starts a new one**
- Rolling updates: deploy new version gradually with **zero downtime**
</v-clicks>
<v-click>
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3           # I want 3 copies
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:v2.1    # Update this → rolling deploy
          ports:
            - containerPort: 3000
```
 
</v-click>
<!--
Deployments are the bread-and-butter of Kubernetes. 90% of what you deploy will be a Deployment.
-->
 
---
layout: two-cols
---
 
## ReplicaSets — Under the Hood
 
A Deployment actually manages a **ReplicaSet** behind the scenes.
 
<v-clicks>
- ReplicaSet ensures *N* copies of a Pod are always running
- Deployment manages ReplicaSets to enable rollouts & rollbacks
- You rarely interact with ReplicaSets directly
</v-clicks>
<v-click>
```
Deployment
    └── ReplicaSet (v2)
            ├── Pod 1 ✅
            ├── Pod 2 ✅
            └── Pod 3 ✅
    └── ReplicaSet (v1) — scaled to 0
```
 
</v-click>
::right::
 
<div class="pl-8">
### The Reconciliation Loop
 
<v-click>
Kubernetes constantly runs a **control loop**:
 
```
while true:
  desired = what you declared
  actual  = what's running
  
  if actual != desired:
    fix it
```
 
</v-click>
<v-click>
This is the core philosophy of Kubernetes:
 
> **Declare what you want. Kubernetes makes it so.**
 
</v-click>
</div>
<!--
The reconciliation loop is one of the most important concepts. Kubernetes is always watching and self-healing. This is "declarative infrastructure."
-->
 
---
 
## Services — Stable Networking
 
Pods are ephemeral — their IP addresses change when they restart. **Services** solve this.
 
<v-clicks>
- A Service provides a **stable IP and DNS name** that points to a group of Pods
- Traffic is automatically **load balanced** across healthy Pods
- Pods join/leave a Service based on **labels**
</v-clicks>
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app          # Route to all Pods with this label
  ports:
    - port: 80           # Port the Service listens on
      targetPort: 3000   # Port the Pod container runs on
  type: ClusterIP        # Only accessible inside the cluster
```
<v-click>
**Service types:**
- `ClusterIP` — internal only (default)
- `NodePort` — expose on a port of each node
- `LoadBalancer` — provision a cloud load balancer
</v-click>
<!--
The labels concept is key — this is how Kubernetes connects Services to Pods. It's not by name, it's by label.
-->
 
---
 
## Labels & Selectors — The Glue
 
**Labels** are key-value pairs attached to any Kubernetes object. They're how everything connects.
 
<v-click>
```yaml
# Pod with labels
metadata:
  labels:
    app: payment-service
    env: production
    version: v3.2
```
 
</v-click>
<v-clicks>
- Services use **selectors** to find their Pods via labels
- Deployments use labels to track which Pods they own
- You can query, filter, and operate on objects by label
</v-clicks>
```bash
# List all production pods
kubectl get pods -l env=production
 
# Get a specific service's pods
kubectl get pods -l app=payment-service,version=v3.2
```
<!--
Labels seem simple but are incredibly powerful. They're used everywhere in Kubernetes for connecting and grouping resources.
-->
 
---
layout: section
---
 
## Going Deeper
### The Control Plane & Advanced Concepts
 
---
layout: two-cols
---
 
## The Control Plane
 
The brain of Kubernetes. Four core components:
 
<v-clicks>
**API Server**
- The front door to Kubernetes
- Everything goes through it (including `kubectl`)
**etcd**
- Distributed key-value store
- The single source of truth — stores all cluster state
</v-clicks>
::right::
 
<div class="pl-8 pt-8">
<v-clicks>
**Scheduler**
- Watches for new Pods with no Node assigned
- Picks the best Node based on resource availability
**Controller Manager**
- Runs the reconciliation loops
- One controller per resource type (Deployment controller, Node controller, etc.)
</v-clicks>
<v-click>
```
You → kubectl → API Server → etcd
                    ↓
              Scheduler
              Controller Manager
                    ↓
              Worker Nodes
```
 
</v-click>
</div>
<!--
You don't need to manage these — managed Kubernetes (EKS, GKE, AKS) handles the control plane for you. But knowing what's under the hood is valuable.
-->
 
---
 
## Namespaces — Virtual Clusters
 
**Namespaces** let you divide one physical cluster into multiple virtual environments.
 
<v-clicks>
- Resources in different namespaces are isolated from each other
- Great for separating `dev`, `staging`, `production` on the same cluster
- Or separating different teams/projects
</v-clicks>
```bash
# Create a namespace
kubectl create namespace staging
 
# Deploy into a specific namespace
kubectl apply -f deployment.yaml -n staging
 
# List pods in all namespaces
kubectl get pods --all-namespaces
```
<v-click>
```
Cluster
├── namespace: default      ← where things go if you don't specify
├── namespace: production
├── namespace: staging
└── namespace: kube-system  ← Kubernetes internal components
```
 
</v-click>
<!--
kube-system is worth mentioning — it's where core k8s services like CoreDNS and kube-proxy run.
-->
 
---
layout: two-cols
---
 
## ConfigMaps & Secrets
 
Separate your **configuration** from your container images.
 
**ConfigMap** — non-sensitive configuration
 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "postgres.default.svc"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
```
 
::right::
 
<div class="pl-8">
**Secret** — sensitive data (base64 encoded)
 
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=  # base64
  API_KEY: c2VjcmV0a2V5
```
 
<v-click>
Mount them as environment variables:
 
```yaml
envFrom:
  - configMapRef:
      name: app-config
  - secretRef:
      name: app-secrets
```
 
</v-click>
<v-click>
> ⚠️ Secrets are only base64 encoded by default — not encrypted. Use tools like **Sealed Secrets** or **Vault** for real security.
 
</v-click>
</div>
<!--
This is a common interview question: what's the difference between a ConfigMap and a Secret? The answer is: Secrets are base64 encoded and meant for sensitive data, ConfigMaps are for plain config.
-->
 
---
 
## Ingress — Traffic from the Outside World
 
A **Service** exposes Pods inside the cluster. **Ingress** exposes services to the internet.
 
<v-clicks>
- Ingress is a routing layer — it maps HTTP/HTTPS traffic to Services
- One Ingress can route to **many services** based on path or hostname
- Requires an **Ingress Controller** to be installed (e.g. nginx-ingress)
</v-clicks>
<v-click>
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```
 
</v-click>
<!--
Ingress is how the real world hits your cluster. Without it, your services are unreachable from outside.
-->
 
---
layout: two-cols
---
 
## Persistent Volumes
 
Containers are stateless by default — data is lost when a Pod restarts.
 
**Persistent Volumes (PV)** and **Persistent Volume Claims (PVC)** solve this.
 
<v-clicks>
- **PV** — a piece of storage provisioned in the cluster
- **PVC** — a *request* for storage by a Pod
- Kubernetes matches PVCs to available PVs
</v-clicks>
::right::
 
<div class="pl-8 pt-4">
```yaml
# Claim storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
 
```yaml
# Use it in a Pod
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: db-storage
volumeMounts:
  - mountPath: "/var/lib/postgresql"
    name: data
```
 
</div>
<!--
Databases running in Kubernetes need persistent volumes. Without them, your Postgres data would vanish every time a Pod restarts.
-->
 
---
 
## Health Checks — Liveness & Readiness Probes
 
Kubernetes can automatically detect and respond to unhealthy containers.
 
<v-click>
**Liveness Probe** — Is the container alive? Kill and restart it if not.
 
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 15   # Wait 15s before first check
  periodSeconds: 10         # Check every 10s
```
 
</v-click>
<v-click>
**Readiness Probe** — Is the container ready to receive traffic?
 
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```
 
</v-click>
<v-click>
> A Pod failing readiness is **removed from the Service load balancer** but not killed. Perfect for graceful startup.
 
</v-click>
<!--
This is one of the more practical features students will use. Health checks are essential for zero-downtime deployments.
-->
 
---
layout: two-cols
---
 
## Resource Requests & Limits
 
Tell Kubernetes how much CPU and memory your containers need.
 
```yaml
resources:
  requests:
    memory: "128Mi"   # Minimum guaranteed
    cpu: "250m"       # 250 millicores = 0.25 CPU
  limits:
    memory: "512Mi"   # Maximum allowed
    cpu: "500m"       # Will be throttled above this
```
 
::right::
 
<div class="pl-8 pt-4">
<v-clicks>
**Why this matters:**
 
- **Requests** → used by the Scheduler to pick the right Node
- **Limits** → enforced at runtime by the OS
- Without limits, one buggy container can starve the whole Node
**CPU is compressible** — it gets throttled
**Memory is not** — the container gets **OOMKilled**
 
```bash
# See resource usage
kubectl top pods
kubectl top nodes
```
 
</v-clicks>
</div>
<!--
OOMKilled (Out Of Memory Killed) is something every Kubernetes engineer will encounter. Always set memory limits.
-->
 
---
 
## Helm — The Package Manager for Kubernetes
 
Deploying a real app to Kubernetes means many YAML files. **Helm** manages them as a single package.
 
<v-clicks>
- A **Chart** is a collection of YAML templates for a complete application
- **Values** let you customize a chart without editing the templates
- Huge ecosystem: install Postgres, Redis, nginx with one command
</v-clicks>
```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
 
# Install PostgreSQL with custom values
helm install my-postgres bitnami/postgresql \
  --set auth.postgresPassword=mysecret \
  --set primary.persistence.size=20Gi
 
# Upgrade a release
helm upgrade my-postgres bitnami/postgresql --set image.tag=16.2
 
# Roll back to previous version
helm rollback my-postgres 1
```
<v-click>
Think of Helm as **npm/pip for Kubernetes** — it manages dependencies and versions.
 
</v-click>
<!--
Helm is not strictly part of Kubernetes, but you'll use it constantly in real projects. It's the de-facto standard for packaging Kubernetes applications.
-->
 
---
layout: center
class: text-center
---
 
## The Kubernetes Ecosystem
 
<div class="text-sm text-gray-400 mb-6">Kubernetes is the foundation — a rich ecosystem builds on top</div>
<div class="grid grid-cols-4 gap-4 text-sm text-left">
<v-clicks>
<div class="bg-blue-900/50 rounded p-3">
<strong>Managed K8s</strong><br/>
EKS (AWS)<br/>GKE (Google)<br/>AKS (Azure)
</div>
<div class="bg-green-900/50 rounded p-3">
<strong>Observability</strong><br/>
Prometheus<br/>Grafana<br/>Jaeger
</div>
<div class="bg-purple-900/50 rounded p-3">
<strong>CI/CD</strong><br/>
Argo CD<br/>FluxCD<br/>Tekton
</div>
<div class="bg-orange-900/50 rounded p-3">
<strong>Service Mesh</strong><br/>
Istio<br/>Linkerd<br/>Cilium
</div>
<div class="bg-red-900/50 rounded p-3">
<strong>Storage</strong><br/>
Rook/Ceph<br/>Longhorn<br/>OpenEBS
</div>
<div class="bg-yellow-900/50 rounded p-3">
<strong>Security</strong><br/>
OPA/Gatekeeper<br/>Falco<br/>Sealed Secrets
</div>
<div class="bg-teal-900/50 rounded p-3">
<strong>Packaging</strong><br/>
Helm<br/>Kustomize<br/>Carvel
</div>
<div class="bg-pink-900/50 rounded p-3">
<strong>Serverless</strong><br/>
Knative<br/>KEDA<br/>OpenFaaS
</div>
</v-clicks>
</div>
<!--
This is the CNCF (Cloud Native Computing Foundation) landscape in miniature. The ecosystem is massive — don't feel overwhelmed. Kubernetes itself is the foundation everything else builds on.
-->
 
---
layout: two-cols
---
 
## Putting It All Together
 
A complete request in a Kubernetes-powered app:
 
```
Browser
  ↓
Load Balancer (cloud)
  ↓
Ingress Controller
  ↓
Service (stable DNS)
  ↓
Pod (your container)
  ↓
Service (database)
  ↓
Pod (Postgres) ← PersistentVolume
```
 
::right::
 
<div class="pl-8">
### What Kubernetes gives you:
 
<v-clicks>
- ✅ **Self-healing** — crashed Pods restart automatically
- ✅ **Scaling** — add replicas with one command
- ✅ **Rolling deploys** — zero-downtime updates
- ✅ **Service discovery** — DNS for every service
- ✅ **Load balancing** — built in
- ✅ **Config management** — ConfigMaps & Secrets
- ✅ **Resource isolation** — namespaces
- ✅ **Portability** — runs anywhere
</v-clicks>
</div>
<!--
Circle back to the orchestration problem from Part 1. Tick off each problem Kubernetes solves.
-->
 
---
layout: center
class: text-center
---
 
## Key `kubectl` Commands
 
```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
 
# Describe a resource (detailed info + events)
kubectl describe pod my-pod-xyz
 
# View logs
kubectl logs my-pod-xyz
kubectl logs -f my-pod-xyz   # follow (like tail -f)
 
# Execute a command inside a container
kubectl exec -it my-pod-xyz -- /bin/sh
 
# Apply a YAML file
kubectl apply -f deployment.yaml
 
# Scale a deployment
kubectl scale deployment my-app --replicas=5
 
# Check cluster resource usage
kubectl top nodes
kubectl top pods
```
<!--
These are the commands students will use 80% of the time. `kubectl get`, `describe`, `logs`, `exec`, and `apply` are the daily drivers.
-->
 
---
layout: end
class: text-center
---
 
# Summary
 
<div class="grid grid-cols-2 gap-8 text-left mt-8">
<div>
### The Journey
- Monoliths → Microservices → Containers
- Containers created an **orchestration problem**
- Kubernetes solves it with **declarative configuration**
</div>
<div>
### Core Concepts
- **Cluster** → Nodes → **Pods** → Containers
- **Deployments** manage replicas + rollouts
- **Services** provide stable networking
- **Ingress** exposes to the internet
- **ConfigMaps/Secrets** manage configuration
</div>
</div>
<div class="mt-8 text-gray-400">
**Next Steps:** Try [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io) locally — get a cluster running in minutes
 
</div>
<!--
End with a call to action. Encourage students to get hands-on. Kubernetes is much easier to understand once you've run `kubectl get pods` yourself.
-->
 
