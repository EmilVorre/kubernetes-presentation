---
theme: frankfurt
title: Kubernetes - From Chaos to Orchestration
titleTemplate: "%s"
author: 'Emil Vorre'
date: '05/06/2006'
infoLine: true
highlighter: shiki
lineNumbers: true
transition: slide-left
mdc: true
---

# Kubernetes
## From Monoliths to Orchestration

A practical introduction for software engineers

<style>
/*
  The Frankfurt nav bar is position:absolute top:0 and ~3.5rem tall (two rows).
  The theme's h1 uses `padding: 5.5rem 3rem 1rem 3rem` as a shorthand, so any
  padding-top longhand without !important gets overridden. We push the layout
  content area down by the nav bar height, then compensate the h1 negative
  margin so the blue banner still bleeds to the very top.
*/
.slidev-layout:not(.center):not(.cover):not(.intro) {
  padding-top: 3.5rem;
}
.slidev-layout:not(.center):not(.cover):not(.intro) h1 {
  margin-top: -3.5rem !important;
  padding-top: 6rem !important;
}
</style>

<!--
- Open with audience context: this is for engineers who ship apps, not cluster admins.
- Promise the arc: why Kubernetes exists, how core primitives work, and what to try next.
- Suggested intro line: "Kubernetes is less about containers and more about operating software safely at scale."
-->

---
layout: section
section: Why Kubernetes?
---

# Why Does Kubernetes Exist?

<!--
- Frame this section as a historical problem-solution journey.
- Tell people to look for the pattern: every new abstraction solves one pain and introduces another.
-->

---

# The Monolith Era

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

<!--
- Emphasize monoliths are not "bad"; they are often the best starting architecture.
- Give a relatable example: one repo, one pipeline, one release train.
- Transition: "So where does it start hurting?"
-->

---

# The Cracks Start to Show

As systems grow, monoliths become painful.

- Small changes require full redeploys
- Hard to scale only one hot path
- Bigger codebase slows feature work
- A bug in one area can impact everything

<!--
- Use a concrete scenario: tiny auth fix forces a full redeploy of checkout.
- Highlight coupling cost: bigger blast radius and slower lead time.
- Transition to microservices as an organizational as much as technical response.
-->

---

# Enter Microservices

Break one large app into smaller independent services.

- Service per domain (auth, orders, payments)
- Independent deploys per service
- Better team ownership boundaries
- Scale only what needs scaling

```text
API Gateway -> Auth / Product / Order / Payment Services
```

<!--
- Define microservices in plain language: split by business capability, not random technical layers.
- Call out the key win: independent deployability for teams.
- Caution: this increases distributed-systems complexity.
-->

---

# New Powers, New Problems

Microservices solve one problem and create another.

- Many services to deploy and monitor
- More network communication between components
- Harder release coordination
- More operational complexity

<!--
- This is the "distributed systems tax" slide.
- Mention practical pain: service discovery, retries, observability, deployment coordination.
- Bridge statement: "We solved code coupling, now we need runtime coordination."
-->

---

# Containers to the Rescue (Briefly)

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

<!--
- Explain Docker's true value: packaging + environment consistency.
- Point out what Docker does not do by itself: scheduling, self-healing, safe rollouts.
- Transition line: "Containers are the unit; orchestration is the system."
-->

---
layout: center
class: text-center
---

# The Orchestration Problem

At scale, containerized apps must:

- restart failed workloads
- scale up/down with demand
- route traffic correctly
- roll out updates safely
- handle config and secrets

**This is exactly what Kubernetes solves.**

<!--
- Slow down here; this is the problem statement Kubernetes answers.
- Stress operations outcomes: reliability, elasticity, and safer change management.
- Invite audience to map each bullet to incidents they've seen in production.
-->

---
layout: center
class: text-center
---

# Kubernetes

<Item title="Definition">
An open-source system for automating deployment, scaling, and management of containerized applications.
</Item>

Originally created at Google, now maintained by CNCF.

<!--
- Give the one-sentence definition confidently; this is your anchor quote.
- Clarify CNCF stewardship as signal of ecosystem maturity and vendor neutrality.
-->

---
layout: section
section: How It Works
---

# How Kubernetes Works

<!--
- Set expectation: we now move from motivation to mechanisms.
- Encourage mental model over memorizing every object kind.
-->

---

# The Big Picture: A Cluster

Kubernetes runs as a **cluster** of machines.

- Control Plane = decision making
- Worker Nodes = run containers
- You interact through `kubectl`

```text
You -> kubectl -> Control Plane -> Worker Nodes
```

<!--
- Describe this as a control loop: you declare intent, cluster converges toward it.
- Clarify `kubectl` talks to API Server, not directly to containers.
-->

---

# Pods — The Smallest Unit

<Item title="Pod">
The smallest deployable unit. One or more containers sharing network and storage, with a single cluster IP. Pods are ephemeral.
</Item>

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

<!--
- Repeat key idea: schedule Pods, not raw containers.
- Mention ephemeral nature: Pods can be replaced at any time; don't store state inside them.
- Briefly explain why multi-container Pod exists (sidecar pattern).
-->

---

# Deployments — Don't Run Pods Directly

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

<!--
- Emphasize this as day-1 best practice: always use higher-level controllers.
- Explain rollout/rollback as safety rails for production changes.
- Optional line: "A Pod is cattle, Deployment is the rancher."
-->

---

# ReplicaSets and Reconciliation

Deployments use ReplicaSets under the hood.

- Desired state: 3 Pods
- Actual state: 2 Pods
- Controller creates the missing Pod

<Item title="Reconciliation Loop">
Kubernetes continuously compares desired state to actual state and acts to close the gap. This loop runs forever, for every resource.
</Item>

<!--
- Define reconciliation clearly: controllers continuously close the gap between intent and reality.
- This is the core Kubernetes concept; many later features are variations of this loop.
-->

---

# Services — Stable Networking

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

<!--
- Clarify why Service exists: Pod IPs are disposable, Service identity is stable.
- Mention DNS naming convention briefly (`service.namespace.svc`).
- Distinguish internal Service access from external exposure (next with Ingress).
-->

---

# Labels and Selectors — The Glue

Labels connect Kubernetes resources.

- Services find target Pods
- Deployments track managed Pods
- Operators filter resources precisely

```bash
kubectl get pods -l env=production
kubectl get pods -l app=payment-service,version=v3.2
```

<Item title="Best Practice">
Establish a consistent label schema early: <code>app</code>, <code>env</code>, <code>version</code>, <code>team</code>. Clean labels make debugging fast.
</Item>

<!--
- Tell audience labels are the relational model of Kubernetes.
- Practical advice: enforce consistent label taxonomy early (app, env, version, team).
-->

---
layout: section
section: Going Deeper
---

# Going Deeper
## Control Plane and Advanced Concepts

<!--
- Announce this as "operator mindset" section: what matters in real production clusters.
-->

---

# The Control Plane

<Item title="API Server">Cluster entrypoint — every interaction goes through here.</Item>
<Item title="etcd">Distributed key-value store — the source of truth for all cluster state.</Item>
<Item title="Scheduler">Assigns Pods to nodes based on resources and constraints.</Item>
<Item title="Controller Manager">Runs the reconciliation loops for every resource type.</Item>

<!--
- Quick mapping: API server = front door, etcd = source of truth, scheduler = placement brain.
- Keep it conceptual; avoid overloading with internals unless asked.
-->

---

# Namespaces — Virtual Clusters

Namespaces partition one physical cluster.

- Separate `dev`, `staging`, `production`
- Isolate teams and projects
- Avoid naming collisions

```bash
kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
kubectl get pods --all-namespaces
```

<!--
- Use this to explain multi-tenant hygiene in a shared cluster.
- Mention namespaces are isolation boundaries for names/resources, not full security boundaries by themselves.
-->

---

# ConfigMaps and Secrets

Keep configuration outside container images.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: info
  FEATURE_X: "true"
```

<Item title="Warning">
Secrets are base64-encoded by default — not encrypted. Combine with RBAC, encryption at rest, and an external secret manager for real security.
</Item>

<!--
- Explain "build once, configure per environment" principle.
- Note that Kubernetes Secrets are base64-encoded objects; real security needs RBAC + encryption-at-rest + secret manager integration.
-->

---

# Ingress — External Traffic

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

<!--
- Clarify that Ingress resource needs an Ingress Controller to actually work.
- Useful phrasing: Service is internal traffic, Ingress is north-south entrypoint.
-->

---

# Persistent Volumes

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

<!--
- Distill PV/PVC model: app asks for storage claim, platform binds it to provisioned volume.
- Mention common stateful examples: databases, queues, artifact stores.
-->

---

# Health Checks — Liveness and Readiness

Probes help route traffic only to healthy Pods.

<Item title="Liveness Probe">Should the container be restarted? Failure → container is killed and restarted.</Item>
<Item title="Readiness Probe">Can it receive traffic right now? Failure → Pod removed from Service load balancer, but not killed.</Item>

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

<!--
- This is a frequent production footgun; emphasize probe correctness.
- Rule of thumb: readiness protects users, liveness protects process health.
-->

---

# Resource Requests and Limits

Define resource intent per container.

- Requests reserve CPU/memory for scheduling
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

<Item title="Common Failure">Memory limit too low → OOMKill loops. Always measure before setting limits.</Item>

<!--
- Explain scheduling behavior: requests drive placement; limits constrain runtime consumption.
- Mention common failure: memory limit too low -> OOMKill loops.
-->

---

# Helm — Package Manager for Kubernetes

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

<!--
- Position Helm as reusable packaging + templating, not replacement for Kubernetes concepts.
- Warn gently about overusing `--set`; encourage versioned values files for teams.
-->

---
layout: center
class: text-center
---

# The Kubernetes Ecosystem

Kubernetes is the platform core with a large ecosystem:

- **Managed K8s:** EKS, GKE, AKS
- **Observability:** Prometheus, Grafana, Jaeger
- **Delivery:** Argo CD, Flux, Tekton
- **Service Mesh:** Istio, Linkerd

<Item title="Advice">Start with the core. Add ecosystem tools only when you have a real problem they solve.</Item>

<!--
- Reinforce that Kubernetes is a platform nucleus; surrounding tools solve observability, delivery, policy.
- Encourage starting simple: don't adopt ecosystem tools until pain justifies complexity.
-->

---
layout: section
section: Beyond the Basics
---

# Beyond the Basics
## Podman, kubeconfig, and CI/CD

<!--
- Quick detour into three things you'll meet outside of "pure" Kubernetes concepts.
- Podman as a Docker alternative, kubeconfig as the cluster credential file, and reusable CI/CD workflows.
-->

---

# Podman — A Docker Alternative

Podman is a daemonless, drop-in replacement for Docker.

- Same command line: `podman build`, `podman run`, `podman ps`
- **No background daemon** — each command is its own process
- **Rootless by default** — better security posture
- Great fit for CI runners and locked-down workstations

```bash
alias docker=podman
docker build -t my-app:0.1 .
docker run --rm -p 3000:3000 my-app:0.1
```

<Item title="Works with Kubernetes too">
Tools like <code>kind</code> and <code>minikube</code> can use Podman as their container runtime instead of Docker. Set <code>KIND_EXPERIMENTAL_PROVIDER=podman</code> and you're done.
</Item>

<!--
- Mention Docker Desktop's licensing change pushed many teams to Podman.
- Same CLI surface area means almost zero re-learning curve.
- Rootless containers genuinely matter on multi-tenant build agents.
-->

---

# kubeconfig — How `kubectl` Finds Your Cluster

Every `kubectl` command reads a file called **kubeconfig**, by default at `~/.kube/config`.

It contains three things stitched together:

- **Clusters** — API server URL + CA cert
- **Users** — credentials (token, cert, OIDC)
- **Contexts** — a named pairing of cluster + user + namespace

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context production
```

<Item title="Working with multiple clusters">
You can merge configs with <code>KUBECONFIG=~/.kube/config:~/.kube/staging</code> or use a tool like <code>kubectx</code> / <code>kubens</code> to switch contexts and namespaces in one keystroke.
</Item>

<!--
- This is the file that managed clusters give you when you run `aws eks update-kubeconfig` or `gcloud container clusters get-credentials`.
- kind also writes its context here automatically when you create a cluster.
- Most production incidents involving "wrong cluster" boil down to checking `current-context`.
-->

---

# Auto-Deploy from CI

Drop a kubeconfig into your GitHub repo as a **secret**, and any workflow can deploy to your cluster.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure kubectl
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBE_CONFIG }}" > $HOME/.kube/config
      - run: kubectl apply -f k8s/
```

<Item title="Real-world example: Trainyard">
<a href="https://github.com/EmilVorre/trainyard">EmilVorre/trainyard</a> uses this exact pattern to spin up an <strong>ephemeral preview environment for every pull request</strong> — a fresh namespace, its own URL, torn down automatically when the PR closes.
</Item>

<!--
- Stress that the kubeconfig in secrets pattern is what unlocks GitOps and PR-based deploys.
- Trainyard is the concrete "what does this enable?" example: full multi-service preview environments per PR.
- Mention key feature: every PR gets pr-42.preview.yourdomain.com, automatically torn down on close.
- Underline that this is built on standard Kubernetes + Helm — no proprietary platform needed.
-->

---
layout: center
---

# Reusable GitHub Workflows

Once you can build and ship a container, the next step is **automating the path from commit to cluster**.

<Item title="EmilVorre/reuseable-workflows">
A collection of reusable GitHub Actions workflows you can drop into any project — including a Kubernetes packaging workflow that builds, tags, and publishes container images ready for deployment.
</Item>

[github.com/EmilVorre/reuseable-workflows](https://github.com/EmilVorre/reuseable-workflows)

```yaml
jobs:
  package:
    uses: EmilVorre/reuseable-workflows/.github/workflows/package.yml@main
    with:
      image-name: my-app
      tag: ${{ github.sha }}
```

<!--
- Position this as the natural next step after the hands-on workshop.
- Highlight the packaging workflow specifically — it's the bridge between "I built a Docker image" and "my cluster has a new version running".
- Reusable workflows mean you don't copy-paste 200 lines of YAML across every repo.
- Encourage attendees to star it and try it on their own project.
-->

---
layout: section
section: Wrap Up
---

# Wrap Up
## Putting It All Together

<!--
- Signal we're closing the loop: tie every concept back into one coherent picture.
- Close with outcomes language: resilience, repeatability, and operational confidence.
-->

---

# A Typical Request Flow

```text
Browser
  -> Load Balancer
  -> Ingress
  -> Service
  -> Pod (application)
  -> Service
  -> Pod (database) + PersistentVolume
```

<!--
- Walk the request path step by step, naming the K8s primitive at each hop.
- Each hop maps to a primitive we covered earlier in the talk.
-->

---

# What Kubernetes Gives You

- **Self-healing** — Pods restart automatically when they fail
- **Scaling** — horizontal scaling on demand
- **Rolling deploys** — zero-downtime updates and easy rollbacks
- **Service discovery** — built-in DNS for every Service
- **Load balancing** — across all healthy Pods of a Service
- **Config + secret management** — config lives outside the image
- **Resource isolation** — requests, limits, and namespaces
- **Portability** — same manifests run on any cluster, anywhere

<!--
- Read this as a closing statement; this is the value proposition of Kubernetes.
- Tie each bullet back to a concept covered earlier in the talk.
-->

---
layout: center
---

# Key `kubectl` Commands

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

<Item title="Debugging Flow">get → describe → logs → exec. This sequence resolves ~90% of cluster issues.</Item>

<!--
- Suggest a practical debugging sequence: get -> describe -> logs -> exec.
- Mention `kubectl top` requires metrics-server, so availability may vary by cluster.
-->

---
layout: section
section: Hands-On
---

# Task 1
## Install kubectl & kind

<!--
- Tell everyone to open a terminal now — they should follow along.
- Remind them kind needs Docker Desktop (or Podman) running in the background.
-->

---

# Prerequisites

Before installing, make sure **Docker** is running on your machine.

<Item title="Docker Desktop">
Download from https://www.docker.com/products/docker-desktop — available for macOS, Windows, and Linux. kind uses it to spin up cluster nodes as containers.
</Item>

Verify Docker is ready:

```bash
docker version
```

<!--
- If Docker isn't installed, kind will install but fail to create clusters.
- Docker Desktop is the easiest path on Mac and Windows; on Linux native Docker works fine.
-->

---

# Install on macOS

Using **Homebrew** (recommended):

```bash
brew install kubectl
brew install kind
```

No Homebrew? Use the direct binaries:

```bash
# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-darwin-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind
```

<!--
- Homebrew is the two-command path most attendees will use.
- Direct binary install works the same on Apple Silicon — swap amd64 for arm64.
-->

---

# Install on Linux

Debian / Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install -y kubectl
```

```bash
# kind binary (works on all distros)
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind
```

<!--
- Debian/Ubuntu apt path is verbose but reliable.
- Arch / Fedora users: `pacman -S kubectl` and `kind` is in the AUR; `dnf install kubectl kind`.
- The kind binary install is universal across distros.
-->

---

# Install on Windows

Using **winget** (built into Windows 10/11):

```powershell
winget install Kubernetes.kubectl
winget install Kubernetes.kind
```

Using **Chocolatey**:

```powershell
choco install kubernetes-cli
choco install kind
```

Using **Scoop**:

```powershell
scoop install kubectl
scoop install kind
```

After installing, open a **new** terminal so PATH updates take effect.

<!--
- winget is the zero-friction option — no extra package manager to install.
- If they're on Windows and winget fails, Chocolatey or Scoop are well-known alternatives.
- Remind Windows users to run the terminal as a normal user, not Administrator.
-->

---

# Verify the Installation

Run these on any platform:

```bash
kubectl version --client
kind version
```

You should see version numbers for both — no errors.

<Item title="Expected output (example)">
Client Version: v1.32.x<br/>
kind version 0.27.x
</Item>

<!--
- If kubectl says "command not found", PATH wasn't updated — close and reopen the terminal.
- If kind version fails, same fix.
- Windows users may need to open a new PowerShell session after winget installs.
-->

---

# Create Your First Cluster

```bash
# Spin up a single-node cluster
kind create cluster --name my-first-cluster

# Confirm the cluster is running
kubectl cluster-info
kubectl get nodes
```

Expected output from `kubectl get nodes`:

```text
NAME                           STATUS   ROLES           AGE   VERSION
my-first-cluster-control-plane Ready    control-plane   30s   v1.32.x
```

<Item title="Clean up when done">
kind delete cluster --name my-first-cluster
</Item>

<!--
- First `kind create cluster` pulls a Docker image (~700 MB) so it may take a minute.
- kubectl get nodes should show one node in Ready state.
- Mention that by default kind configures kubectl automatically — no manual kubeconfig needed.
-->

---
layout: intro
class: text-center
---

# Summary

## The Journey

Monoliths → Microservices → Containers

Containers created orchestration complexity

Kubernetes solves it with declarative operations

## Core Concepts

Cluster → Nodes → Pods

Deployments manage replicas and updates

Services provide stable networking

Ingress handles external routing

ConfigMaps, Secrets, and PVCs manage runtime needs

**Next step:** run a local cluster with [minikube](https://minikube.sigs.k8s.io) or [kind](https://kind.sigs.k8s.io)

<!--
- Recap in one line: Kubernetes turns operational intent into continuously enforced reality.
- End with action: spin up local cluster and deploy one app with Deployment + Service + Ingress.
- Invite Q&A based on their current production pain points.
-->
