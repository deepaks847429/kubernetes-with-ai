# Day 2 — Why Bare Pods Are a Trap: Deployments & Services

**Date:** 2026-09-21 · **Phase:** 1 (Core Kubernetes) · **Status:** ✅ done

## Goals for today (all met)
1. ✅ Prove a **bare pod** does not self-heal.
2. ✅ Meet the **Deployment** — self-healing + scaling via reconciliation.
3. ✅ Meet the **Service** — a stable address that solves Day 1's ephemeral/unreachable pod-IP problem.

---

## The arc: bare pod → Deployment → Service

### 1. Bare pods are a trap
- Recreated the lab cluster (deleted at end of Day 1) — the "disposable lab" habit: one config file, ~2 min.
- Created a bare pod: `kubectl run web --image=nginx`.
- **Deleted it → it stayed dead forever.** A bare pod has no controller watching it. Delete it, crash it, or lose its node → gone. Unacceptable in production.

### 2. Deployments = self-healing + scaling
- `kubectl create deployment web --image=nginx --replicas=3`
- Learned the **three-layer hierarchy**:
  ```
  Deployment "web"  →  ReplicaSet "web-<hash>"  →  Pods "web-<hash>-<rand>"
  (I declare intent)   (keeps N pods alive)         (the actual containers)
  ```
- **Delete a pod → a new one appears in seconds.** The ReplicaSet reconciles actual → desired. This is self-healing.
- **Scaling is the same loop:** `kubectl scale deployment web --replicas=5` (grows) / `--replicas=2` (shrinks). There is no separate up/down command — I declare the target count and Kubernetes computes the diff. `--replicas=0` is legal (scale to zero; the Deployment object stays).

### 3. Services = stable front door
- Problem: 3 pods, 3 different **ephemeral** IPs — clients need one address that never changes and load-balances.
- `kubectl expose deployment web --port=80` → a Service with a **stable ClusterIP** and DNS name `web`.
- `kubectl get endpoints web` → the Service maps to the current pod IPs, updated automatically as pods heal/scale.
- Reached the app **by name, no IP**, from inside the cluster:
  `kubectl run tmp --image=busybox:1.36 --rm -it --restart=Never -- wget -qO- http://web`
- `kubectl describe svc web` → **`Selector: app=web`** — the Service finds pods by **label**, same label the ReplicaSet uses.
- Browser via `kubectl port-forward service/web 8080:80` → http://localhost:8080.

---

## Key commands learned

| Command | What it does |
|---|---|
| `kubectl create deployment web --image=nginx --replicas=3` | Create a self-healing, scalable app |
| `kubectl get deploy` / `get rs` / `get pods` | See the Deployment → ReplicaSet → Pod hierarchy |
| `kubectl scale deployment web --replicas=N` | Declare a new desired pod count (up or down) |
| `kubectl expose deployment web --port=80` | Create a Service (stable ClusterIP + DNS) in front of the pods |
| `kubectl get svc` / `get endpoints web` | See the stable IP and the live pod-IP backend list |
| `kubectl describe svc web` | Reveals the `Selector` (label-based routing) |
| `kubectl run tmp --image=busybox:1.36 --rm -it --restart=Never -- <cmd>` | Throwaway pod to test from *inside* the cluster |
| `kubectl port-forward service/web 8080:80` | Tunnel a local port to a Service |

## Concepts that clicked

- **Imperative vs Declarative:** Docker says "run this now" (one-shot). Kubernetes says "always keep this true" (declared desired state, continuously reconciled). This is the core difference.
- **Reconciliation / self-healing:** a controller drags actual state toward desired state, forever. Healing and scaling are the *same* loop.
- **Deployment → ReplicaSet → Pod** hierarchy — I manage the Deployment; it manages the rest.
- **Labels are the universal glue:** ReplicaSets keep `app=web` pods alive; Services route to `app=web` pods. Neither uses pod IPs. Pods are cattle (identified by label), not pets (identified by IP).
- **Service = stable ClusterIP + DNS + load balancing** over a dynamic set of pods; solves Day 1's "pod IPs are ephemeral and unreachable."
- **ClusterIP is internal-only** — getting traffic from *outside* the cluster is a separate concern (NodePort/LoadBalancer/Ingress → Day 3).

## Questions I can now answer in an interview

- Why not just create pods directly? (bare pods don't self-heal — no controller)
- What's the difference between a Deployment, a ReplicaSet, and a Pod?
- What is reconciliation / self-healing? Give an example. (delete a pod, ReplicaSet recreates it)
- Imperative vs declarative — what does Kubernetes being "declarative" mean?
- How does a Service know which pods to route to? (label selector, not IPs)
- Why do we need Services at all? (stable address + load balancing over ephemeral pods)
- How do pods talk to each other? (by Service DNS name, e.g. `web`, resolved by CoreDNS)

## Key realizations (not a breakage today — a clean concept day)

- The "aha": a Deployment isn't "a fancy pod" — it's a *promise* Kubernetes keeps. I stopped thinking in "start a container" and started thinking in "declare what should be true."
- Labels quietly wire everything together; once I saw `Selector: app=web` in both the ReplicaSet and the Service, the whole system clicked.

## Next (Day 3)
1. **Getting traffic from OUTSIDE the cluster:** NodePort → LoadBalancer → **Ingress** (the promised follow-up).
2. **Stop being imperative — go declarative with YAML:** write a Deployment + Service as `.yaml` and `kubectl apply -f` them. This is how real work (and GitOps) is done.
3. Possibly: labels & selectors deeper, and reading `kubectl get -o yaml` to see what Kubernetes filled in.
