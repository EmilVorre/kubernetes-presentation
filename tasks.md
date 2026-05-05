# Kubernetes Hands-On Tasks

A short, ~30 minute lab that takes you from an empty cluster to a running, scaled, configured, health-checked application. Each task builds on the previous one — go in order.

**Prerequisites** (covered in the slides):
- `kubectl` installed
- `kind` installed
- Docker running

> All commands work on macOS, Linux, and Windows. Where Windows differs, alternative syntax is shown.

---

## Task 0 — Create your cluster (2 min)

If you didn't create one during the talk:

```bash
kind create cluster --name workshop
kubectl cluster-info
kubectl get nodes
```

You should see one node with `STATUS: Ready`. Done? Move on.

---

## Task 1 — Run your first Pod (3 min)

The fastest way to launch something in Kubernetes:

```bash
kubectl run hello --image=nginx:1.25 --port=80
kubectl get pods
kubectl describe pod hello
```

**Look at the output of `describe`** — find the `Events:` section at the bottom. This is the timeline of what Kubernetes did: scheduled → pulled image → started container.

Now delete it; we won't be using `kubectl run` again:

```bash
kubectl delete pod hello
```

**Lesson:** `kubectl run` is fine for quick experiments, but in real life we use Deployments.

---

## Task 2 — Create a Deployment (5 min)

Create a file called `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

Apply it and watch it come up:

```bash
kubectl apply -f deployment.yaml
kubectl get pods -l app=web
kubectl get deployment web
```

**Now break things on purpose.** Delete one of the Pods:

```bash
# Pick any pod name from `kubectl get pods` output
kubectl delete pod <pod-name>
kubectl get pods -l app=web
```

A new Pod appeared within seconds. **That's reconciliation in action** — the ReplicaSet noticed actual replicas (2) didn't match desired (3) and created a new one.

---

## Task 3 — Expose with a Service (5 min)

Pods have IPs, but they change. A Service gives you a stable address.

Create `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

Apply it and inspect:

```bash
kubectl apply -f service.yaml
kubectl get service web
kubectl describe service web
```

**Look at the `Endpoints:` line** — those are the Pod IPs the Service is load-balancing to. They were discovered automatically via the `app: web` label selector.

Now hit the Service from your laptop using port-forwarding:

```bash
kubectl port-forward service/web 8080:80
```

Open another terminal and run:

```bash
curl http://localhost:8080
# Windows PowerShell alternative:
# Invoke-WebRequest http://localhost:8080
```

You should see the Nginx welcome HTML. Stop the port-forward with `Ctrl+C` when done.

---

## Task 4 — Scale and roll out an update (5 min)

### Scale up

```bash
kubectl scale deployment web --replicas=5
kubectl get pods -l app=web -w
```

`-w` watches in real time. Press `Ctrl+C` once all five Pods are `Running`.

### Roll out a new version

Edit `deployment.yaml` and change the image:

```yaml
        image: nginx:1.27
```

Apply it and watch the rolling update:

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/web
kubectl get pods -l app=web
```

Kubernetes replaced the old Pods one batch at a time. No downtime.

### Roll back

Pretend the new version was broken:

```bash
kubectl rollout undo deployment/web
kubectl rollout status deployment/web
```

You're back on the previous version. **This is your safety net for every production deploy.**

---

## Task 5 — ConfigMap and environment variables (5 min)

Configuration belongs *outside* the image. Create `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  WELCOME_MESSAGE: "Hello from Kubernetes!"
  LOG_LEVEL: "info"
```

Apply it:

```bash
kubectl apply -f configmap.yaml
kubectl get configmap web-config -o yaml
```

Now wire it into the Deployment. Edit `deployment.yaml` and add an `env` block under the container:

```yaml
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
          env:
            - name: WELCOME_MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: web-config
                  key: WELCOME_MESSAGE
            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: web-config
                  key: LOG_LEVEL
```

Apply and verify:

```bash
kubectl apply -f deployment.yaml
kubectl get pods -l app=web
# Pick any running pod
kubectl exec -it <pod-name> -- env | grep -E "WELCOME_MESSAGE|LOG_LEVEL"
```

You should see your config values inside the container — without rebuilding the image.

---

## Task 6 — Health checks (4 min)

Probes let Kubernetes know when a Pod is healthy and when to stop sending it traffic.

Edit `deployment.yaml` and add probes under the container spec:

```yaml
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 2
            periodSeconds: 5
```

Apply and confirm Pods are still healthy:

```bash
kubectl apply -f deployment.yaml
kubectl describe pod <pod-name>
```

Look at the `Conditions:` section — `Ready: True` means the readiness probe passed.

### Simulate a failure

Break one Pod's nginx process and watch Kubernetes restart it:

```bash
# Pick any running web pod
kubectl exec -it <pod-name> -- nginx -s stop
kubectl get pods -l app=web -w
```

Within a few seconds the Pod's `RESTARTS` count will go up and it returns to `Ready`. **Self-healing in action.**

Press `Ctrl+C` to stop watching.

---

## Task 7 — Clean up (1 min)

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete -f configmap.yaml

# Or nuke the whole cluster
kind delete cluster --name workshop
```

---

## Bonus tasks (if you have time)

### Bonus A — Run two services and route between them

Create a second Deployment + Service called `api` running `hashicorp/http-echo` with arguments `["-text=hello from api"]`. From inside any `web` Pod, `curl http://api/` and observe Kubernetes' built-in DNS resolving the Service name.

### Bonus B — Add resource requests and limits

Add a `resources:` block to the container spec with `requests: { cpu: 100m, memory: 64Mi }` and `limits: { cpu: 200m, memory: 128Mi }`. Apply, then run `kubectl describe pod` and find the resource section.

### Bonus C — Persistent storage

Define a `PersistentVolumeClaim` requesting `1Gi` and mount it into a Pod at `/data`. Write a file inside, delete the Pod, recreate it, and verify the file still exists.

---

## Cheat sheet

```bash
# Inspect
kubectl get pods,services,deployments
kubectl describe <resource> <name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- /bin/sh

# Apply / delete
kubectl apply -f <file>.yaml
kubectl delete -f <file>.yaml

# Rollout
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>

# Cluster
kubectl get nodes
kubectl cluster-info
kind delete cluster --name workshop
```

---

## What you just learned

| Task | Concept |
|---|---|
| 1 | Pods are the unit of execution |
| 2 | Deployments + reconciliation = self-healing |
| 3 | Services give Pods stable network identity |
| 4 | Rolling updates and rollbacks are first-class operations |
| 5 | Config lives outside the image |
| 6 | Probes turn Kubernetes into a health monitor |

That's the core of Kubernetes. Everything else is a variation on these patterns.

---

## Task 8 — Deploy one of your own projects (30 min)

Take any small project you have lying around and run it on Kubernetes. The goal is to wire together everything from the previous tasks for *your own* code.

**Scope rules** so you finish in 30 minutes:

- Pick a **single-process web app** (HTTP server). No databases, no queues, no auth — strip it down.
- It must **listen on a single port** and respond to a healthcheck path (or just `/`).
- If it needs config, expose it through **environment variables**, not files.
- Ignore TLS, persistence, and inbound DNS for this exercise.

If you don't have something ready, use this **5-line fallback project** in any folder:

```js
// server.js
const http = require('http');
const port = process.env.PORT || 3000;
const msg  = process.env.GREETING || 'Hello from my app!';
http.createServer((_, res) => res.end(msg + '\n')).listen(port);
console.log(`listening on ${port}`);
```

---

### Step 1 — Write a Dockerfile (5 min)

In your project folder, create a `Dockerfile`. Adjust the base image and start command for your stack.

**Node.js example:**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Python example:**

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**Go example (multi-stage):**

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app .

FROM gcr.io/distroless/static
COPY --from=build /app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```

For the fallback `server.js` above, no `package.json` is needed — drop the `npm install` line.

---

### Step 2 — Build and load the image into kind (5 min)

This is the step people forget. `kind` runs in Docker but **does not see your local images automatically**:

```bash
docker build -t my-app:0.1 .
kind load docker-image my-app:0.1 --name workshop
```

Verify the image is in the cluster's nodes:

```bash
docker exec -it workshop-control-plane crictl images | grep my-app
```

You should see `my-app:0.1` listed.

> If you skip `kind load`, your Pod will fail with `ErrImagePull` because Kubernetes will try to pull from a registry it can't reach.

---

### Step 3 — Write the manifests (10 min)

Create `my-app.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
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
          image: my-app:0.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000  # change to your app's port
          env:
            - name: GREETING
              value: "Hello from Kubernetes!"
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 2
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000  # change to your app's port
  type: ClusterIP
```

**Three things you must edit for your project:**
- `image:` if you tagged differently
- `containerPort:` and `targetPort:` to your app's port
- `env:` block with whatever env vars your app reads (or remove it)

---

### Step 4 — Deploy and verify (5 min)

```bash
kubectl apply -f my-app.yaml
kubectl get pods -l app=my-app
kubectl logs -l app=my-app --tail=20
```

Wait for `STATUS: Running` and `READY: 1/1`. If something looks wrong, see the troubleshooting table below.

Reach the app from your laptop:

```bash
kubectl port-forward service/my-app 8080:80
```

In another terminal:

```bash
curl http://localhost:8080
# Windows PowerShell:
# Invoke-WebRequest http://localhost:8080
```

You should see your app's response. **You're now running your own code on Kubernetes.**

---

### Step 5 — Iterate (5 min)

Try at least one of these to feel the dev loop:

- **Change a response string in your code**, rebuild, reload, redeploy:
  ```bash
  docker build -t my-app:0.2 .
  kind load docker-image my-app:0.2 --name workshop
  kubectl set image deployment/my-app app=my-app:0.2
  kubectl rollout status deployment/my-app
  ```
- **Scale to 5 replicas** and `curl` repeatedly — you're load-balancing your own app.
- **Crash one Pod** with `kubectl delete pod <name>` and watch it come back.

---

### Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `ErrImagePull` / `ImagePullBackOff` | Image not loaded into kind | Re-run `kind load docker-image my-app:0.1 --name workshop` |
| `CrashLoopBackOff` | App crashes on start | `kubectl logs <pod>` — usually a missing env var or wrong start command |
| Pod `Running` but `0/1 Ready` | Readiness probe failing | Check probe `path` and `port` match what your app actually serves |
| `port-forward` returns nothing | Wrong `targetPort` | Confirm the port your app listens on inside the container |
| `connection refused` from inside cluster | App bound to `127.0.0.1` instead of `0.0.0.0` | Make your server listen on all interfaces |

---

### Stretch goals (if you finish early)

- Add a **ConfigMap** and pull `GREETING` from it instead of inline.
- Add **resource requests and limits** to keep your container honest.
- Write a **liveness probe** that hits a different endpoint than the readiness probe.
- Expose the app via **Ingress** — install [ingress-nginx for kind](https://kind.sigs.k8s.io/docs/user/ingress/) and try a hostname-based route.

When you're done:

```bash
kubectl delete -f my-app.yaml
```
