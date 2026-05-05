# Kubernetes Presentation — Speaker Script

Word-for-word script for the slides in `slides-clean.md`. Read it out loud at a conversational pace.

**Time estimate:** ~38–45 minutes for the full talk **before** the hands-on workshop, plus ~30 minutes for participants to do the tasks. Total session ≈ **70–75 minutes**.

A breakdown by section is at the bottom of this file.

---

## Slide 1 — Title: Kubernetes, From Monoliths to Orchestration

Hey everyone, thanks for being here. Today we're going to talk about Kubernetes — but not the way you usually see it introduced, which is "here are twenty YAML files, good luck."

Instead, I want to walk you through *why* Kubernetes exists, *what problem it actually solves*, and *how its core building blocks fit together*. By the end, my goal is that when you hear words like "Pod", "Deployment", "Service", or "Ingress", you have a real mental model — not just a vocabulary list.

This talk is aimed at software engineers — people who write and ship applications. You don't need to be a cluster admin or a DevOps specialist to get value from it.

One sentence to set the tone: *Kubernetes is less about containers, and more about operating software safely at scale.*

After the talk, we'll spend about thirty minutes doing hands-on exercises together — so keep your laptop ready.

Let's start at the beginning.

---

## Slide 2 — Part 1: Why Does Kubernetes Exist?

Before we look at any Kubernetes concept, I want to spend a few minutes on the history — because Kubernetes didn't appear out of nowhere. It's the answer to a chain of problems we ran into as an industry.

I'd like you to watch for one pattern as we go through this section: *every new abstraction we adopted solved one pain, and then created a new one*. Kubernetes is just the latest step in that chain.

---

## Slide 3 — The Monolith Era

Most software, historically, started as a monolith. One codebase, one deployable artifact, one database behind it. Authentication, products, orders, billing — all of it living together in the same process.

And honestly? That's a great place to start. A monolith is easy to build, easy to run locally, easy to deploy, and easy to reason about. For a small team or an early-stage product, a monolith is often the *correct* architecture. I want to be clear about that — monoliths are not a mistake. They're a phase.

So if monoliths are so good, why didn't we stay there? Well... they start to hurt as the system grows. Let's look at where.

---

## Slide 4 — The Cracks Start to Show

As your team and your codebase grow, a few patterns start showing up.

First: *small changes need full redeploys*. You change one line in the auth module, and you have to rebuild and redeploy the entire application — including checkout, search, and everything else. The blast radius of every change is the whole system.

Second: *you can't scale just one part*. Maybe your search endpoint is getting hammered, but your admin panel is idle. With a monolith, if you want more search capacity, you scale the whole monolith — including all the parts that didn't need scaling.

Third: *the codebase slows the team down*. Build times grow, test suites grow, merge conflicts grow, onboarding gets harder.

And fourth: *one bug can take everything down*. A memory leak in one feature can crash the process that's serving every other feature.

So at some point, teams start asking: "what if we split this thing up?"

---

## Slide 5 — Enter Microservices

That's where microservices come in. The idea is simple: take that one big application and break it into smaller, independent services, usually split along business domains. So instead of one app, you have an auth service, a product service, an orders service, a payments service, and so on.

This unlocks four important things:

*One*, each service has clear ownership — usually one team per service.

*Two*, you can deploy services independently. The payments team doesn't have to wait for the search team's release window.

*Three*, you can scale only the services that need scaling.

*Four*, the codebase per service stays smaller and more focused, which means faster iteration.

Sounds great, right? Well... yes. But.

---

## Slide 6 — New Powers, New Problems

Here's the catch: when you replace one big application with twenty small services, you've essentially traded *code complexity* for *operational complexity*.

Now you have many things to deploy, many things to monitor, many things to secure. Your services have to talk to each other over the network — which means you suddenly have to care about retries, timeouts, circuit breakers, observability, and service discovery.

Releases get harder to coordinate. If service A depends on a new field in service B, you have to think about deployment order. And if any one of those services is unhealthy, your overall product might still be broken even though "everything is technically running."

This is sometimes called the "distributed systems tax." Microservices solved code coupling, but now we have a runtime coordination problem.

So before we go to Kubernetes, there's one more piece of the puzzle that has to fall into place — and that's containers.

---

## Slide 7 — Containers to the Rescue (Briefly)

Docker, and containers in general, gave us two huge wins.

First, *packaging*. Your application and all its dependencies — the runtime, the libraries, the system packages — ship together as a single, immutable image. No more "it works on my machine."

Second, *consistency*. The same image runs the same way on your laptop, in CI, in staging, and in production. Plus containers start in milliseconds, not minutes like traditional VMs.

If you look at the Dockerfile on screen, this is basically the entire mental model: pick a base image, copy your code in, declare your dependencies, expose a port, define how to start. That's it. It's a tiny, portable, runnable unit of software.

But — and this is the important part — *Docker by itself doesn't run your production system*. Docker can build and start one container. It doesn't decide *which machine* the container should run on, it doesn't restart it if it crashes, it doesn't roll out a new version safely, it doesn't load-balance across replicas, and it doesn't handle secrets or configuration.

Containers are the *unit*. We still need a *system* that orchestrates those units across many machines. And that's where Kubernetes finally enters the picture.

---

## Slide 8 — The Orchestration Problem

So when you actually run containerized microservices in production, you very quickly run into a list of operational problems that you don't want to solve by hand.

You need workloads to *restart automatically* when they fail.

You need to *scale up and down* with demand — both horizontally and ideally automatically.

You need *traffic to be routed correctly* to healthy instances, not to crashed ones.

You need *safe rollouts* — you don't want every deploy to be a "fingers crossed" moment.

And you need to manage *configuration and secrets* in a structured way, not by editing files on servers.

If you try to script all of that yourself, you essentially end up writing your own bad version of an orchestrator. Kubernetes is what the industry collectively decided to standardize on instead.

---

## Slide 9 — Kubernetes: One Sentence Definition

Here's the one-sentence definition I want you to remember:

*Kubernetes is an open-source system for automating the deployment, scaling, and management of containerized applications.*

That's it. Three jobs: deploy, scale, manage.

A bit of history: it was originally created at Google, based on internal systems they'd been using for over a decade to run their own services at planetary scale. Then in 2014 they open-sourced it, and today it's stewarded by the Cloud Native Computing Foundation, the CNCF — which is important, because it means no single vendor controls it. That neutrality is one of the reasons it became the industry standard.

Okay — that's the *why*. Now let's look at *how*.

---

## Slide 10 — Part 2: How Kubernetes Works

This is the core mechanics section. We're going to go from "what is a cluster" all the way down to the building blocks you'll touch every day: Pods, Deployments, Services, and the glue that holds them together.

I want to give you mental models, not exhaustive specs. If you walk away understanding how the pieces fit, you'll be able to read the docs for any individual feature on your own.

---

## Slide 11 — The Big Picture: A Cluster

Kubernetes runs as a *cluster* — a group of machines working together as if they were one big computer.

There are two kinds of machines in that cluster.

The *Control Plane* is the brain. It makes decisions: where should things run, what's the current state, what needs to happen next.

The *Worker Nodes* are the muscles. They actually run your containers.

You, as a user, almost never log into any of those machines directly. Instead, you talk to the cluster through a command-line tool called `kubectl`. You send `kubectl` a description of what you want — "I want three replicas of this app running" — and the cluster figures out how to make that true.

This is the most important mental shift in Kubernetes: you describe *desired state*, and the cluster *converges toward it*. You don't tell it the steps; you tell it the goal.

---

## Slide 12 — Pods: The Smallest Unit

Now, the smallest deployable unit in Kubernetes is *not* a container. It's a *Pod*.

A Pod is a wrapper around one or more containers that should always live together, on the same node, sharing the same network and storage. Most of the time, a Pod has exactly one container — that's the common case. The case where a Pod has multiple containers is usually the *sidecar pattern*: for example, your app container plus a logging agent, or plus a service mesh proxy.

A few things to remember about Pods. They get their own IP address inside the cluster. They share storage volumes between their containers. And — this is important — they are *ephemeral*. A Pod can be destroyed and recreated at any time. You should never treat a specific Pod as permanent, and you should never store state inside a Pod that you can't afford to lose.

The YAML on screen is the simplest possible Pod: an Nginx container, exposing port 80. In real life, you almost never write Pod YAML directly — you use a higher-level object that *manages* Pods for you. Which is the next slide.

---

## Slide 13 — Deployments: Don't Run Pods Directly

So here's a rule of thumb: in production, you almost never create Pods directly. You create *Deployments*, and Deployments create Pods on your behalf.

Why? Because a Deployment gives you four very important guarantees.

*One*, it maintains a desired number of replicas. If you say "I want three", and one Pod dies, Kubernetes will start a new one to bring the count back to three.

*Two*, it does *rolling updates*. When you change the image version, Kubernetes will gradually replace the old Pods with new ones, while keeping the service available the whole time.

*Three*, it supports *rollbacks*. If a new version is broken, you can revert to the previous version with a single command.

*Four*, it replaces unhealthy Pods automatically.

The three commands at the bottom are basically the entire day-to-day workflow: you `apply` your YAML, you watch the rollout status, and if anything goes wrong, you `undo`. That's the safety rail.

A line I like: *a Pod is cattle; a Deployment is the rancher.*

---

## Slide 14 — ReplicaSets and Reconciliation

Under the hood, a Deployment uses something called a *ReplicaSet*, whose only job is to make sure the right number of Pods exist.

The mechanism is so important that it deserves its own slide. Imagine you've said "I want three Pods." The ReplicaSet checks: how many are actually running right now? If it sees two, it creates one more. If it sees four, it deletes one. If it sees three, it does nothing.

That loop runs continuously, forever, for every resource in the cluster.

This is called *reconciliation*, and it is the single most important concept in Kubernetes. Almost every feature in Kubernetes is some variation of: *"compare the desired state to the actual state, and close the gap."*

Once you internalize that, the whole system suddenly makes sense.

---

## Slide 15 — Services: Stable Networking

Okay, so we have Pods that come and go, with IP addresses that change. How do other parts of the system actually *talk* to them reliably?

That's what a *Service* is for. A Service is a stable network identity in front of a set of Pods.

When you create a Service, it gets a fixed virtual IP and a fixed DNS name inside the cluster. Other Pods can talk to that DNS name and not worry about which actual Pods are behind it. The Service load-balances requests across all the healthy Pods that match its selector.

Look at the YAML: the key part is the `selector`. It says "this Service routes traffic to any Pod with the label `app: my-app`." That's how Services know which Pods to send traffic to. And `type: ClusterIP` means this Service is only reachable *inside the cluster* — that's the default, and it's what you want for service-to-service communication.

For exposing things to the outside world, we'll get to Ingress in a couple of slides.

---

## Slide 16 — Labels and Selectors: The Glue

Speaking of selectors — let's talk about the labeling system, because it's the connective tissue of the whole platform.

Labels are just key-value pairs you attach to any Kubernetes object. Things like `app: payment-service`, `env: production`, `version: v3.2`, `team: billing`.

That's it — they're just metadata. But everything in Kubernetes uses them to find related resources. A Service finds its target Pods by labels. A Deployment tracks its Pods by labels. Monitoring tools, network policies, autoscalers — they all filter by labels.

This is essentially the *relational model* of Kubernetes. Instead of hard-coded references, things find each other through label queries.

Practical advice: pick a consistent labeling scheme *early*, and document it. Things like `app`, `env`, `version`, `team`, `tier`. If your cluster has clean labels, debugging is fast. If it doesn't, debugging is suffering.

---

## Slide 17 — Going Deeper: Control Plane and Advanced Concepts

Alright, we've covered the foundational objects: Pods, Deployments, Services, labels. That's enough to deploy real applications.

In this section, I want to give you the next layer — the things you'll absolutely meet in any real production environment. We're going to look at the control plane components, namespaces, configuration, ingress, storage, health checks, resource limits, and Helm.

This is the *operator mindset* layer. Not deep operations expertise, but enough that nothing in a production cluster will feel like a black box anymore.

---

## Slide 18 — The Control Plane

So what's actually inside the brain of Kubernetes? Four core components.

The *API Server* is the front door. Every interaction — `kubectl`, dashboards, controllers, other components — goes through the API Server. It's the only thing that talks directly to the database.

*etcd* is the database — a distributed key-value store. It holds the entire desired state of the cluster. If you lose etcd, you lose the cluster's memory.

The *Scheduler* decides *which node* a new Pod should run on, based on available resources, constraints, affinities, and so on.

The *Controller Manager* runs all the reconciliation loops we talked about — the things that constantly compare desired state to actual state and act to close the gap.

You very rarely interact with these components directly. But it helps a lot to know they exist, because when something weird happens in a cluster, the answer is usually in one of these four.

---

## Slide 19 — Namespaces: Virtual Clusters

Now, in real life, you don't usually have one cluster per environment. You often have one big cluster shared by many teams or environments. *Namespaces* are how you partition it.

A namespace is essentially a virtual cluster *inside* the physical cluster. You can have a `dev` namespace, a `staging` namespace, a `production` namespace — or one namespace per team, per project, per tenant.

Namespaces give you scoping for names — two services can both be called `api` if they're in different namespaces — and they give you a unit for applying quotas, role-based access control, and network policies.

One thing I want to be honest about: a namespace by itself is *not* a strong security boundary. It's an organizational and policy boundary. If you need real isolation between tenants, you combine namespaces with RBAC, network policies, and sometimes separate clusters.

---

## Slide 20 — ConfigMaps and Secrets

Next: configuration. The principle here is simple — *build your image once, configure per environment.* You should never bake environment-specific values into your container image.

Kubernetes gives you two objects for that.

*ConfigMaps* hold non-sensitive configuration: log levels, feature flags, hostnames, timeouts, that kind of thing. The example on screen is just a key-value store you can mount into a Pod as environment variables or as files.

*Secrets* are for sensitive data: API keys, database passwords, certificates. Same shape as ConfigMaps, but treated differently by the platform.

One thing I want to be very clear about: by default, Kubernetes Secrets are *base64-encoded*, not encrypted. They're not magically secure. In any serious environment you'd combine them with RBAC, encryption at rest in etcd, and ideally an external secret manager — something like HashiCorp Vault, AWS Secrets Manager, or Sealed Secrets — so the actual secret never lives in plaintext in your repo.

---

## Slide 21 — Ingress: External Traffic

So we said Services handle internal traffic. What about *external* traffic — actual users on the internet hitting your app?

That's where *Ingress* comes in. Ingress is an object that describes how external HTTP and HTTPS traffic should be routed *into* your cluster, to your Services.

It does three main things: it routes traffic by hostname and path — so `app.example.com` goes to one Service, `api.example.com` goes to another. It terminates TLS, so you handle HTTPS in a centralized way. And it gives you a clean, declarative way to expose internal Services to the outside world.

One thing that sometimes confuses people: an Ingress *resource* on its own does nothing. You also need an *Ingress Controller* — like Nginx, Traefik, or a cloud load balancer integration — running in your cluster to actually implement the routing rules. The resource is the spec; the controller is the implementation.

A good way to remember it: *Service is east-west traffic, Ingress is north-south traffic.*

---

## Slide 22 — Persistent Volumes

Containers, by default, are temporary. When a Pod restarts, anything written to its filesystem is gone. That's fine for a stateless web app — but it's a disaster for a database.

Kubernetes solves this with two objects: *PersistentVolume* and *PersistentVolumeClaim*.

A *PersistentVolume*, or PV, is the actual storage resource — a piece of disk somewhere, provided by the cluster.

A *PersistentVolumeClaim*, or PVC, is your *request* for storage. You say "I want twenty gigabytes, read-write, single-node access" — and Kubernetes finds or creates a matching PV and binds them together.

Your Pod then mounts the PVC, and from inside the container, it just looks like a directory. But it's durable — if the Pod restarts or moves to another node, the data stays.

Anything stateful — databases, message queues, artifact stores, file uploads — uses this pattern.

---

## Slide 23 — Health Checks: Liveness and Readiness

Health checks are one of the most underrated features in Kubernetes — and one of the most common sources of production incidents when they're configured wrong. So please pay attention here.

There are two probes you should know.

A *liveness probe* answers the question: "is this container still alive, or is it stuck and should be restarted?" If the liveness probe fails repeatedly, Kubernetes kills the container and starts a new one.

A *readiness probe* answers a different question: "is this container ready to accept traffic *right now*?" If the readiness probe fails, the Pod is removed from its Service's load balancer — but the container is *not* killed. It's just temporarily taken out of rotation.

The rule of thumb I want you to remember: *readiness protects your users; liveness protects your process.*

A common mistake is to point both probes at the same heavy endpoint, like `/healthz`, that hits the database. That can make a slow database take down your entire deployment. So design these endpoints carefully — readiness can be stricter, liveness should usually be cheap and minimal.

---

## Slide 24 — Resource Requests and Limits

Every container should declare how much CPU and memory it needs. There are two numbers.

*Requests* are what your container is *guaranteed* to get. The scheduler uses requests to decide which node has room to run your Pod. If you don't set requests, the scheduler is essentially flying blind.

*Limits* are the *cap* — the maximum your container is allowed to consume. If a container exceeds its memory limit, it gets killed — that's the famous OOMKilled event you'll see in `kubectl describe`.

A few practical tips. Setting requests too low means the scheduler thinks your app is small, packs too many things on a node, and they all start fighting for CPU. Setting limits too low means your app gets killed under load. And not setting limits at all on memory means one buggy Pod can take down a whole node.

This is one of those areas where a little bit of measurement upfront — actually looking at the metrics — saves a lot of pain later.

---

## Slide 25 — Helm: Package Manager for Kubernetes

Once you've been using Kubernetes for a while, you'll notice you're writing similar YAML over and over for every app: a Deployment, a Service, an Ingress, a ConfigMap, maybe a PVC. And then you want slightly different versions per environment.

*Helm* is the package manager that solves this.

A Helm *chart* is a templated bundle of Kubernetes manifests. You combine the chart with a `values.yaml` file that holds your environment-specific settings, and Helm renders the final manifests and applies them.

The commands on screen show the typical flow: you add a chart repository, you install a chart with some custom values, you upgrade it when you need to change something, and you can roll back to a previous release if something goes wrong.

Two practical notes. *One*, Helm is great for installing third-party software — Postgres, Redis, monitoring stacks. *Two*, when teams write their own charts, prefer versioned `values.yaml` files in git over a long chain of `--set` flags. The flags are fine for demos; values files are auditable.

---

## Slide 26 — The Kubernetes Ecosystem

A quick reality check before we get into the next section. Kubernetes itself is just the *core platform* — the kernel, if you will. Around it there's a huge ecosystem of tools that solve adjacent problems.

If you don't want to manage the cluster yourself, every major cloud has a *managed Kubernetes* offering — EKS on AWS, GKE on Google Cloud, AKS on Azure. For most teams, that's the right starting point.

For *observability*, you'll typically see Prometheus for metrics, Grafana for dashboards, and something like Jaeger or Tempo for distributed tracing.

For *delivery*, GitOps tools like Argo CD and Flux let you treat your Git repository as the source of truth and auto-sync changes into the cluster. Tekton handles pipelines.

For *service-to-service communication* at scale — mTLS, traffic policies, retries — you have service meshes like Istio and Linkerd.

A piece of advice: don't adopt all of these on day one. Kubernetes has a reputation for complexity, and a lot of that reputation comes from teams turning every problem into "and now we'll add another tool." Start with the core. Add ecosystem pieces only when you have a real problem they solve.

---

## Slide 27 — Beyond the Basics: Podman, kubeconfig, and CI/CD

Before we wrap up the conceptual part, I want to give you three more things you'll definitely run into in real life — but that don't always show up in introductions to Kubernetes.

The first is *Podman*, an alternative to Docker. The second is *kubeconfig*, the file that tells `kubectl` which cluster to talk to. And the third is how you take all of this and *automate it* with CI/CD — so deployments happen on every commit, not on every developer typing `kubectl apply` from their laptop.

These three together close the loop between "I learned Kubernetes" and "my team actually uses Kubernetes in production."

---

## Slide 28 — Podman: A Docker Alternative

Quick word on Podman, because some of you probably can't run Docker Desktop.

Podman is essentially a drop-in replacement for Docker. The CLI is identical — `podman build`, `podman run`, `podman ps`. You can literally `alias docker=podman` and most scripts and Dockerfiles just work.

But there are two important differences under the hood.

*One*, it's *daemonless*. Docker has a long-running root daemon that everything talks to. Podman doesn't — every command is its own process. That's nicer for CI runners, locked-down environments, and anywhere you don't want a privileged background service.

*Two*, it's *rootless by default*. You can run containers without root privileges, which is genuinely better security on multi-tenant build agents.

A lot of teams switched to Podman after Docker Desktop changed its licensing for larger companies. It also plays nicely with Kubernetes — both `kind` and `minikube` can use Podman as their container runtime. Set the environment variable `KIND_EXPERIMENTAL_PROVIDER=podman` and you're done.

So if you ever hit Docker friction, Podman is a real option without giving up anything you've learned.

---

## Slide 29 — kubeconfig: How kubectl Finds Your Cluster

Okay, so we've talked about `kubectl` a lot. But how does `kubectl` actually know which cluster to talk to? The answer is a single file called *kubeconfig*, by default at `~/.kube/config`.

Inside, three things are stitched together.

*Clusters*, which are the API server URL plus the certificate authority that signs its TLS cert.

*Users*, which are the credentials you use — a token, a client certificate, or an OIDC integration.

And *Contexts*, which are named pairings of one cluster, one user, and one default namespace.

In real life, you'll have multiple contexts: maybe one for your local kind cluster, one for staging, one for production. The three commands you'll use most are `kubectl config get-contexts`, to list everything you have access to; `current-context`, to see where you're pointed right now; and `use-context`, to switch.

A pro tip: when you run `aws eks update-kubeconfig` or `gcloud container clusters get-credentials`, that's just a fancy way of merging a new entry into this file. Tools like *kubectx* and *kubens* let you switch contexts and namespaces with a single keystroke, and they're worth installing on day one.

Honestly, the most common production incident I've seen that starts with "I don't understand what happened" ends with someone realizing they were running `kubectl delete` against the wrong context. Always check `current-context` before destructive operations.

---

## Slide 30 — Auto-Deploy from CI

Here's where it all comes together. Once you have a kubeconfig file that authenticates against your cluster, you can drop that file into a *GitHub repository secret* and let any GitHub Actions workflow deploy on your behalf.

The workflow on screen is basically the entire pattern. Check out the code, write the kubeconfig from the secret to disk, run `kubectl apply`. That's it. From this primitive, you can build *any* deployment automation you want — push to main and deploy to production, push to a branch and deploy to a preview environment, label a PR and spin up an isolated test stack.

To make that last one really concrete: I built a project called *Trainyard* — link's right there in the call-out. It uses exactly this pattern to spin up an *ephemeral preview environment for every pull request*. You add a `preview` label to a PR, GitHub Actions builds the image, deploys it to a fresh namespace, and posts a comment with a unique URL like `pr-42.preview.yourdomain.com`. When you close the PR, it's torn down automatically. All self-hosted, runs on a tiny VPS, no proprietary platform involved — just standard Kubernetes, standard Helm, and GitHub Actions wired together.

The reason I'm showing you this is that it demonstrates how much *leverage* you get once you understand kubeconfig plus a CI runner. You don't need a special platform. You just need the primitives we've already talked about.

---

## Slide 31 — Reusable GitHub Workflows

One last piece. If you do this for many repositories, you'll quickly notice you're copy-pasting a lot of the same workflow YAML — building a container, tagging it, pushing it to a registry. That gets tedious and error-prone.

GitHub Actions has a feature called *reusable workflows*, where you publish a workflow once and other repos call it with `uses:` plus a few parameters. I maintain a small public collection at `github.com/EmilVorre/reuseable-workflows` that has ready-made workflows for common cases — including a Kubernetes packaging workflow that builds, tags, and publishes container images so they're ready for the deploy step we just looked at.

The four-line snippet on screen is all the YAML it takes to use the packaging workflow in your own repo. You set the image name, you set the tag, and that's it — the heavy lifting lives in the shared workflow.

This is the natural evolution: you start writing your own GitHub Actions, you notice the duplication, you extract reusable workflows, and now adding CI/CD to a new repo takes minutes instead of an afternoon. Feel free to star the repo and use it on your own projects.

---

## Slide 32 — Wrap Up: Putting It All Together

Okay, now we're going to wrap up the conceptual part of the talk and tie everything we've covered into one coherent picture. Then we'll move on to the hands-on workshop.

---

## Slide 33 — A Typical Request Flow

Let me walk you through what happens when a real user hits your application.

It starts with a *browser* opening your URL. DNS resolves to a *Load Balancer* — usually one provided by your cloud provider. That load balancer forwards the request to the *Ingress Controller* running inside your cluster. The Ingress Controller looks at the host and path, matches it against your Ingress rules, and sends the request to the right *Service*. That Service load-balances the request to one of the healthy *Pods* running your application code. That Pod, in turn, talks to its database — which is itself a Pod, fronted by another Service, with data stored on a *PersistentVolume* so it survives Pod restarts.

That's the whole picture. Every hop on that path corresponds to a Kubernetes object we've talked about today. Once you can draw this diagram from memory, you can reason about almost any Kubernetes deployment.

---

## Slide 34 — What Kubernetes Gives You

So when you stitch all of those primitives together, here's what you actually get.

*Self-healing* — Pods restart automatically when they fail.

*Scaling* — horizontal scaling on demand, manually or via the Horizontal Pod Autoscaler.

*Rolling deploys* — zero-downtime updates with built-in rollback.

*Service discovery* — every Service has a DNS name; Pods find each other automatically.

*Load balancing* — across all the healthy Pods of a Service.

*Config and secret management* — your image stays the same across environments.

*Resource isolation* — through requests, limits, and namespaces.

And *portability* — the same manifests run on any cluster, on any cloud, on your laptop.

That's the value proposition. It's not "containers." Containers were already useful. The value is *running them as a coherent, reliable system*.

---

## Slide 35 — Key kubectl Commands

A quick reference of the commands you'll use every day. I won't read each one, but I want to highlight a typical debugging flow because it's useful muscle memory.

When something is wrong, the sequence is usually:

*One*, `kubectl get` — to see what exists. `get pods`, `get deployments`, `get services`.

*Two*, `kubectl describe` — to see *why* something might be wrong. Events, conditions, recent changes. This is where most of the answers actually live.

*Three*, `kubectl logs` — to see what the application itself is saying.

*Four*, `kubectl exec` — only if the previous three didn't tell you enough. Get a shell inside the Pod and look around.

`kubectl top` is useful for quick resource snapshots, but it depends on `metrics-server` being installed in your cluster — so don't be surprised if it doesn't work everywhere out of the box.

If you internalize *get → describe → logs → exec*, you can debug ninety percent of cluster problems. Bookmark this slide; it's your day-one cheat sheet.

---

## Slide 36 — Hands-On: Task 1, Install kubectl & kind

Alright — that's the conceptual part of the talk. Now we're going hands-on.

For the rest of our time together, we're going to install the tools, spin up a real Kubernetes cluster on your laptop, and deploy a real application into it. You'll see the concepts we just talked about — Pods, Deployments, Services, reconciliation, rolling updates — happen on your own screen.

The first thing we need to install is `kubectl`, the command-line tool, and `kind`, which stands for "Kubernetes IN Docker" — it lets you run a complete Kubernetes cluster as a Docker container on your laptop.

Open a terminal now. We'll go through it together.

---

## Slide 37 — Prerequisites

Before we install anything, one thing has to be true: *Docker has to be running*. Kind works by spinning up Kubernetes nodes as Docker containers, so without Docker, kind has nothing to work with.

If you don't have Docker, install Docker Desktop from the URL on screen. It's available for macOS, Windows, and Linux. On Linux you can also use the native Docker engine if you'd rather avoid Desktop.

Once Docker is up, run `docker version` in your terminal. You should see both the client and server reporting versions. If the server section says "cannot connect," start Docker Desktop and try again.

Take a moment now to make sure that command works. We can't continue without it.

---

## Slide 38 — Install on macOS

If you're on macOS, the easiest path is *Homebrew*. Two commands: `brew install kubectl` and `brew install kind`. That's it. Homebrew handles the binary, PATH, and updates.

If you don't have Homebrew, you can install both tools from direct binaries — the commands are on screen. The pattern is the same for both: download, make it executable, move it to `/usr/local/bin`. If you're on Apple Silicon, swap `amd64` for `arm64` in the URLs.

Take 30 seconds and run the install. Tell me when you're done.

---

## Slide 39 — Install on Linux

On Linux, the easiest path depends on your distro.

For Debian or Ubuntu, the official Kubernetes apt repository is the right way. The block on screen looks like a lot, but it's only doing four things: making sure you have HTTPS support, adding the Kubernetes signing key, adding the apt source, and installing `kubectl`.

For *kind*, the binary install works on every distro — Debian, Ubuntu, Fedora, Arch, whatever. Download, chmod, move into `/usr/local/bin`, done.

If you're on Arch, by the way, both `kubectl` and `kind` are in your package manager. Same on Fedora — `dnf install kubectl kind` works.

---

## Slide 40 — Install on Windows

If you're on Windows, you have three good package manager options.

The simplest is *winget*, which ships with Windows 10 and 11. Two commands and you're done.

If you already use *Chocolatey*, the package names are `kubernetes-cli` for `kubectl`, and just `kind` for kind.

If you prefer *Scoop*, the names are `kubectl` and `kind`, same as the binaries.

One important note: after installing, *open a fresh terminal*. Windows package managers update the PATH for new processes, but your existing shell still has the old PATH. Otherwise you'll get "command not found" and think it failed.

---

## Slide 41 — Verify the Installation

Now let's make sure everything is wired up. Run these two commands on any platform: `kubectl version --client` and `kind version`.

You should see version numbers for both, no errors. The exact versions don't matter much — Kubernetes is generally backwards compatible across two minor versions.

If `kubectl` says "command not found," PATH didn't update — open a new terminal session. Same fix for kind. On Windows, that almost always means starting a fresh PowerShell window.

Take a second now to confirm you see the versions. Once everyone's there, we'll create our first cluster.

---

## Slide 42 — Create Your First Cluster

Here's the magic moment. One command: `kind create cluster --name my-first-cluster`.

That will pull a Kubernetes node image — about 700 megabytes the first time, so it'll take a minute — and start a fully functional Kubernetes cluster as a single Docker container on your machine. By the time it finishes, kind has also configured your kubeconfig so `kubectl` is already pointed at the new cluster.

Run `kubectl cluster-info` to confirm the API server is reachable, and `kubectl get nodes` to see your one-node cluster in action. You should see one node in `Ready` state.

Congratulations — you now have a real Kubernetes cluster on your laptop.

When you're ready to clean up later, the command is `kind delete cluster --name my-first-cluster`. But don't delete it yet, because we're going to deploy things into it.

---

## Slide 43 — Summary

Quick recap, and then we're going to dive into the workshop tasks together.

We started with a *journey*. Monoliths gave us simplicity but didn't scale. Microservices gave us scaling but added operational complexity. Containers gave us packaging but didn't orchestrate. Kubernetes filled that last gap with declarative, self-healing operations.

Then we walked through the *core concepts*. Clusters made of Nodes, running Pods. Deployments to manage replicas and updates. Services for stable internal networking. Ingress for external traffic. ConfigMaps, Secrets, and PVCs for the runtime needs of real applications.

We went *beyond the basics* into Podman, kubeconfig, and the CI/CD patterns that turn Kubernetes from "cool toy" into a real production platform.

The *one-line takeaway*: Kubernetes turns operational intent into continuously enforced reality. You declare what you want, and the cluster works to make it true — and to keep it true.

Your *next step* is the workshop tasks I'm about to hand out. They'll take you from your fresh kind cluster to a deployed, scaled, configured, health-checked application — and then to deploying *your own* code. Everything we just talked about, in your hands.

Thanks so much, and let's get into it.

---

# Time Estimate

| Section | Slides | Approx. Speaking Time |
|---|---|---|
| **Title + Part 1: Why** (slides 1–9) | 9 | ~10 min |
| **Part 2: How** (slides 10–16) | 7 | ~8–9 min |
| **Going Deeper** (slides 17–26) | 10 | ~12–13 min |
| **Beyond the Basics** (slides 27–31) | 5 | ~6–7 min |
| **Wrap Up** (slides 32–35) | 4 | ~4–5 min |
| **Hands-On intro** (slides 36–42) | 7 | ~5–8 min (depends on install time) |
| **Summary** (slide 43) | 1 | ~1 min |
| **Total speaking time** | **43 slides** | **~45–55 min** |
| **Hands-on workshop (`tasks.md`)** | — | **~30–45 min** |
| **Total session** | | **~75–100 min** |

## Tuning the length

- **Tight 60-min slot**: Skip slides 14, 19, 22, 24, 28, and one of 30/31. Cut hands-on to Tasks 1–4. Lands around 60 min total.
- **45-min talk only (no workshop)**: Skip the Hands-On install slides (36–42). Lands around 40 min talk + 5 min Q&A.
- **Half-day workshop (3+ hours)**: Run the full talk, then do all of `tasks.md` *including* Task 8 (deploy your own project). That's the 30-min self-deploy task plus the earlier ~30 min lab.

## Pacing notes

- Slides 8 and 9 ("Orchestration Problem" and the definition) are anchor moments — slow down for emphasis.
- Slides 14 (reconciliation) and 23 (probes) are concepts students remember most — invest the extra 30 seconds.
- Slides 30 and 31 (Trainyard, reusable workflows) are demo-friendly — if you have time and a screen, open the GitHub repos live for 30 seconds each.
- Hands-On install slides (36–42) tend to expand because of attendee questions ("my Docker won't start", "winget said X"). Build in a 5-minute buffer.
