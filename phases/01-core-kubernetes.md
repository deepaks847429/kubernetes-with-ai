# Phase 1 — Core Kubernetes

**Duration:** 3–4 weeks · **Why it exists:** This is the body of every interview at every level. The difference between junior and senior answers here is *mechanism*: juniors describe what objects do, seniors describe what the components do to make it happen.

Lab default: a 3-node kind cluster (1 control plane, 2 workers) rebuilt from a checked-in config file.

---

## 1. Architecture — the control loop worldview

Kubernetes is **a database (etcd) fronted by an API server, watched by controllers that reconcile desired vs actual state**. Internalize that sentence; every feature is an instance of it.

- **kube-apiserver:** the only thing that talks to etcd; authn → authz → admission chain; watches.
- **etcd:** Raft consensus (why odd numbers, what quorum loss means), revisions, compaction.
- **kube-scheduler:** filtering then scoring; what "Pending" really means.
- **kube-controller-manager:** the reconcile loop; informers/watch caches (why controllers don't hammer the API).
- **kubelet:** node agent; PodSpecs → CRI calls; probes executed here, not by the control plane.
- **kube-proxy:** Services → iptables/IPVS rules (and the modern answer: eBPF CNIs replacing it).
- **CRI / CNI / CSI:** the three plugin seams. Name a real implementation of each.

**The marquee question — rehearse until fluent:** *"Walk me through everything that happens when you run `kubectl apply -f deployment.yaml`"* — kubectl → API server (authn/authz/admission, incl. mutating/validating webhooks and defaulting) → etcd write → deployment controller creates ReplicaSet → RS controller creates Pods (Pending) → scheduler binds pod to node → kubelet sees the binding via watch → CRI pulls image, CNI wires networking, CSI mounts volumes → containers start → probes gate Ready → endpoint-slice controller adds pod IP → kube-proxy/CNI programs the dataplane. Practice a 1-minute and a 5-minute version.

## 2. Workloads

- **Pod deep-dive:** phases vs container states, init containers, **sidecar containers** (native restartPolicy: Always on init containers — know the modern mechanism, not just the old pattern), ephemeral containers (`kubectl debug`), pause container's job (holds the namespaces), pod lifecycle hooks.
- **Graceful shutdown — the senior favorite:** SIGTERM → preStop hook → terminationGracePeriodSeconds → SIGKILL; endpoints being removed *in parallel* with termination, which is why a small preStop sleep prevents 502s during rollouts.
- **Deployments:** rolling update mechanics (maxSurge/maxUnavailable), revision history, `kubectl rollout undo`, when rollouts get stuck (progressDeadlineSeconds).
- **StatefulSets:** stable identity, ordered ops, headless services + per-pod DNS, volumeClaimTemplates, why "scale down deletes pods but not PVCs".
- **DaemonSets:** node-level agents; how they schedule (tolerations + node affinity, not the scheduler bypass of the old days).
- **Jobs/CronJobs:** completions/parallelism, backoffLimit, podFailurePolicy, concurrencyPolicy, startingDeadlineSeconds, and why idempotency is *your* problem.

## 3. Services & networking

- The K8s network model: every pod has a routable IP, no NAT pod-to-pod. Then: how a **Service** is *not* a proxy process but dataplane rules.
- ClusterIP / NodePort / LoadBalancer / ExternalName; headless services; EndpointSlices.
- **Ingress vs Gateway API:** Gateway API is the modern answer (roles split: GatewayClass/Gateway/HTTPRoute); still know Ingress because the installed base is huge. Run both in lab.
- CoreDNS: service/pod records, the `ndots:5` issue, NodeLocal DNSCache as the fix.
- Service internals question: *"Client pod curls a ClusterIP — trace the packet"* (DNS → VIP → iptables/IPVS DNAT or eBPF → pod IP → CNI routing → conntrack for the return path).
- NetworkPolicy: default-allow world → default-deny namespace pattern; policies are enforced by the CNI, not by Kubernetes itself.

## 4. Configuration & storage

- ConfigMaps/Secrets: env vs volume mounts (volumes update live, env doesn't — restarts needed), immutable configmaps, secret encoding-is-not-encryption, encryption at rest (EncryptionConfiguration).
- Downward API, projected volumes.
- **Storage:** PV/PVC/StorageClass, dynamic provisioning, access modes (RWO vs RWX and what actually supports RWX), reclaim policies (the `Retain` rescue drill), volume expansion, topology-aware provisioning (WaitForFirstConsumer and *why* — zone-affinity deadlocks), CSI snapshots/clones.

## 5. Scheduling & resources

- **Requests vs limits — the most-asked resource question:** requests drive scheduling + CFS weight; limits drive throttling/OOM. QoS classes (Guaranteed/Burstable/BestEffort) and eviction order. The "should we set CPU limits at all?" debate — know both sides.
- nodeSelector → node affinity → pod affinity/anti-affinity → **topologySpreadConstraints** (the modern HA-spread answer).
- Taints/tolerations (repel) vs affinity (attract) — and combining them for dedicated node pools.
- PriorityClasses & preemption; PodDisruptionBudgets (and how a bad PDB blocks `kubectl drain` forever — every operator has lived this).
- Pod overhead, in-place pod resize (resize CPU/memory without restart — check current maturity at refresh time).

## 6. Health & lifecycle

- Liveness vs readiness vs **startup** probes; failure modes of each (liveness restart loops on slow startups → that's what startup probes fix; readiness ≠ liveness — copy-pasting one into the other causes cascading restarts under load).
- Resource pressure: kubelet eviction thresholds, node conditions (MemoryPressure/DiskPressure), what happens to evicted pods.

## 7. RBAC & access (core level)

- authn (certs, tokens, OIDC) vs authz (RBAC) vs admission.
- Role/ClusterRole, RoleBinding/ClusterRoleBinding; ServiceAccounts and projected bound tokens.
- `kubectl auth can-i --as` — the debugging move.
- kubeconfig anatomy: clusters/users/contexts.

## 8. kubectl fluency (interview practicals are timed)

- Imperative generators: `kubectl create deploy/job/cm ... --dry-run=client -o yaml` — never hand-write YAML from scratch in a timed exam.
- `-o jsonpath`/`-o custom-columns`, `--field-selector`, label selectors, `kubectl explain --recursive`, events (`kubectl get events --sort-by=.lastTimestamp`), `kubectl debug node/`, `rollout status/undo`, `port-forward`, `cp`, `top`.

---

## Labs (do all, in order)

1. Deploy a 2-tier app (API + Redis) with probes, resources, anti-affinity, PDB. Kill nodes; watch it survive.
2. Break rollouts: bad image tag, failing readiness probe, too-tight maxUnavailable + PDB. Diagnose each from `kubectl` output only.
3. StatefulSet Postgres: kill pods, prove data survives; scale down/up; do a `Retain` PV rescue.
4. Trace a Service packet: dump iptables rules for one ClusterIP and annotate each chain (KUBE-SERVICES → KUBE-SVC-* → KUBE-SEP-*).
5. Zero-downtime proof: rollout under `hey`/`k6` load with preStop + readiness tuned — 0 non-200s; then remove the preStop and show the 502s.
6. RBAC: build a namespace-admin persona; prove blast radius with `auth can-i`.

## Interview lens

- "Deployment vs StatefulSet — when and why?" (identity + storage, not "databases")
- "Pod stuck Pending / CrashLoopBackOff / ImagePullBackOff — differential diagnosis for each."
- "How does a rolling update achieve zero downtime, exactly? Where can it still drop traffic?"
- "Why does DNS in Kubernetes get slow?" (ndots + UDP conntrack races; NodeLocal DNSCache)
- "Readiness vs liveness — give me a production horror story for misusing each."

## Exit criteria

- [ ] The `kubectl apply` walkthrough, fluent at 1-min and 5-min depth.
- [ ] All six labs done, war stories logged in PROGRESS.md.
- [ ] Can diagnose Pending/CrashLoop/ImagePull/OOMKilled from symptoms in under 5 minutes each.
- [ ] Can write Deployment/Service/Ingress/PVC YAML from memory (or via `--dry-run=client`) fast enough for a timed exam.
