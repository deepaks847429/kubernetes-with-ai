# Day 1 — First Cluster, Node vs Pod, and Two Networking Lessons

**Date:** 2026-09-21 · **Phase:** 0 → 1 (foundations, getting the lab breathing) · **Time:** ~1 session

---

## The one sentence I'm carrying from now on

> **Kubernetes is a database (etcd) fronted by an API server, watched by controllers that continuously reconcile the actual state of the world to match my desired state — with a scheduler placing work onto nodes.**

Everything else is a special case of this.

---

## What I did

1. **Verified my toolbox:** Docker, kubectl (v1.31), kind (v0.20), helm already installed. (k9s missing — will add later. kind is a bit old → ships K8s v1.27; fine for fundamentals, upgrade in Phase 2.)
2. **Started Docker Desktop** — nothing works until the engine is up (kind runs nodes as Docker containers).
3. **Created my first cluster** from the checked-in config:
   ```powershell
   kind create cluster --name lab --config ..\labs\kind-cluster.yaml
   ```
   Result: a **3-node cluster** — `lab-control-plane`, `lab-worker`, `lab-worker2`.
4. **Inspected it three ways** and saw that a "node" is just a Docker container, and Kubernetes' brain is just Linux processes/pods inside the control-plane node.
5. **Deployed my first workload** and watched the scheduler place it:
   ```powershell
   kubectl run web --image=nginx
   kubectl get pods -o wide     # landed on lab-worker2, IP 10.244.2.2
   ```
6. **Reached the pod the proper way** with a port-forward tunnel (after debugging — see war stories):
   ```powershell
   kubectl port-forward web 9090:80
   # browser → http://localhost:9090 → "Welcome to nginx!"
   ```

## Key commands learned

| Command | What it does |
|---|---|
| `kind create cluster --name lab --config <file>` | Build a multi-node cluster from config |
| `kind delete cluster --name lab` | Wipe it clean (disposable labs) |
| `kubectl get nodes` | List the machines in the cluster |
| `kubectl get pods -A` | List all pods in **all** namespaces (`-A`) |
| `kubectl get pods -o wide` | Adds NODE + IP columns — shows *where* a pod runs |
| `kubectl run web --image=nginx` | Create a single (bare) pod |
| `kubectl describe pod <name>` | Full detail; **Events** at the bottom = troubleshooting gold |
| `kubectl logs <pod>` | The container's own logs |
| `kubectl port-forward <pod> 9090:80` | Tunnel a local port into a pod |

## Concepts that clicked

- **Node vs Pod:** a **node** is a machine (the "where", the infrastructure); a **pod** is the smallest unit K8s runs (the "what", the workload). Pods run on nodes; one node hosts many pods.
- **Container ⊂ Pod ⊂ Node:** my app container lives inside a pod, which runs on a node.
- **Control plane vs workers:** etcd, api-server, scheduler, controller-manager run **only** on the control-plane node (the brain). My apps run on **worker** nodes. Kept separate on purpose so a runaway app can't crash the cluster's brain.
- **The scheduler placed my pod**, not me — I asked for a pod, K8s chose the node (`lab-worker2`).
- **Every pod gets its own IP** (mine: `10.244.2.2`). Each node owns a slice of the pod network (`10.244.2.x` = worker2's slice). The CNI (`kindnet`) provides this.
- **Namespaces** partition the cluster; `kubectl get pods` (no flag) only shows the `default` namespace.

---

## 🔥 War stories (the interview gold)

### War story 1 — "I couldn't reach my pod by its IP"
- **Symptom:** browser to `http://10.244.2.2` → `ERR_CONNECTION_TIMED_OUT`.
- **Root cause:** a pod's IP lives on the **internal pod network**, which exists *inside* the cluster. My Windows host isn't on that network, so it has no route → timeout (not a refusal).
- **Lesson:** pod IPs are (1) **not routable from outside** and (2) **ephemeral** — they change every time a pod is recreated. That's *why* **Services** (stable address) and **Ingress** (outside front door) exist. I felt the problem before meeting the fix.

### War story 2 — "The tunnel connected but returned nothing"
- **Symptom:** `curl.exe http://localhost:8080` → `curl: (52) Empty reply from server`; browser → `ERR_EMPTY_RESPONSE`.
- **Investigation (layer by layer):**
  1. `kubectl get pod web` → `1/1 Running` → pod healthy. ✅
  2. `kubectl logs web` → nginx started fine **but zero access-log lines** → requests never reached nginx. 🔑
  3. `curl.exe` also failed → **not** a browser problem; isolated to the port-forward tunnel layer.
- **Reading the error precisely:** curl **52** = connected but empty reply (≠ **7** connection refused, ≠ **28** timeout). The exact error told me the tunnel accepted the connection but failed to carry it to the pod.
- **Root cause:** flaky/stale `kubectl port-forward` (known on Windows/WSL2 + kind).
- **Fix:** restarted the tunnel on a fresh port with explicit IPv4 (`kubectl port-forward web 9090:80`) → nginx served.
- **Lesson:** to localize a failure, eliminate layers one at a time — **pod → tunnel → browser**. Confirm the app is actually serving (logs/access lines) before blaming the network. This *is* the "the app is down, go" interview drill.

---

## Questions I can now answer in an interview

- What's the difference between a node and a pod?
- What runs on the control plane vs the workers, and why keep them separate?
- What decides which node a pod runs on? (the scheduler)
- Why can't I reach a pod by its IP from my laptop? Why don't we use pod IPs directly? (internal network + ephemeral → Services exist)
- How do I quickly reach a pod for debugging? (`kubectl port-forward`)
- How would you tell whether "site not loading" is the app, the network tunnel, or the browser? (check pod status/logs, then curl bypassing the browser, read the exact error)

---

## Next (Day 2)

1. **The delete experiment:** `kubectl delete pod web` → does it come back? (No — it's a bare pod.) Learn *why you almost never create bare pods*.
2. **Deployments:** tell K8s to *keep* N copies running no matter what — self-healing. The thing Kubernetes is actually *for*.
3. **Services:** the stable address that solves War Story 1 (pod IPs being ephemeral/unreachable).

**Housekeeping to do:** install `k9s`; consider upgrading kind for a newer K8s (defer to Phase 2).
