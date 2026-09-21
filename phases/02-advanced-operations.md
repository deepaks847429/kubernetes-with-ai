# Phase 2 — Advanced Operations & Troubleshooting

**Duration:** 3–4 weeks · **Why it exists:** This phase is what separates "used Kubernetes" from "operated Kubernetes." Senior interviews live here: cluster lifecycle, networking internals, autoscaling, observability, and cold-sweat troubleshooting. Do this phase on **kubeadm VMs**, not kind — you need to feel the control plane.

---

## 1. Cluster lifecycle

- **kubeadm anatomy:** what `init` actually does (certs, static pod manifests in `/etc/kubernetes/manifests`, kubeconfigs, bootstrap tokens); static pods vs regular pods (kubelet-managed, API-server-mirrored).
- **PKI:** the CA hierarchy, which component talks to which over which cert, `kubeadm certs check-expiration`, rotating expired certs (the classic year-one outage).
- **etcd operations — non-negotiable for senior roles:** `etcdctl snapshot save/restore` (do a full restore drill), quorum math (3 vs 5 nodes, what 1-of-3 loss vs 2-of-3 loss means), defragmentation, the 8GB default DB cap, watch latency as the canary metric.
- **HA control plane:** stacked vs external etcd, load balancer in front of API servers, leader election in scheduler/controller-manager (only one is active — know how to check).
- **Upgrades:** the skew policy (kubelet may lag apiserver, never lead it), kubeadm upgrade sequence (control plane → nodes), `drain` → upgrade → `uncordon`, and what PDBs do to your drain. Managed-cluster flavor: surge upgrades, node pool blue/green.
- **Node management:** cordon/drain semantics, node leases and the NotReady timeline (who marks NotReady, when pods get evicted — the pod-eviction-timeout / taint-based eviction chain), kubelet garbage collection (images, containers).

## 2. Networking deep dive

- **CNI landscape with opinions:** Cilium (eBPF, kube-proxy replacement, Hubble observability, increasingly the default answer), Calico (BGP option, mature policy), Flannel (simple overlay, no policy). Be able to argue a choice.
- **eBPF vs iptables dataplane:** why per-service iptables chains degrade at scale (O(n) rule traversal, contention on updates) and what eBPF maps change about that.
- Overlay (VXLAN) vs native routing (BGP, cloud route tables); MTU pitfalls.
- **conntrack:** table exhaustion symptoms, UDP DNS races, tuning.
- Service meshes — the honest senior take: what mTLS/retries/traffic-splitting/telemetry buy you, what the sidecar tax costs, and the sidecar-less trend (Istio ambient, Cilium mesh). Know *when you'd say no* to a mesh; interviewers respect that more than feature lists.
- LoadBalancer internals: cloud LB → NodePort → pod, `externalTrafficPolicy: Local` vs `Cluster` (source IP preservation vs imbalance), MetalLB for bare metal.

## 3. Autoscaling — all four axes

- **HPA:** the algorithm (desired = ceil(current × metric/target)), stabilization windows and flapping, scaling on custom/external metrics (prometheus-adapter), behavior policies.
- **VPA:** modes, restart cost, "HPA and VPA on the same CPU metric = fight" — how to combine safely (VPA for requests baseline, HPA on a different signal).
- **Cluster Autoscaler vs Karpenter:** CA scales pre-defined node groups; Karpenter provisions right-sized nodes directly from pod requirements (bin-packing, consolidation, spot handling). Karpenter is the modern answer for AWS-style interviews — lab it if you have cloud access.
- **KEDA:** event-driven scaling (queue depth, Kafka lag, cron), **scale-to-zero**, ScaledObjects/ScaledJobs. Bridges into the AI phase (queue-based GPU scaling).

## 4. Observability

- **Metrics:** Prometheus architecture (pull, service discovery, exporters), PromQL working set (rate, histogram_quantile, sum by), recording rules, Alertmanager routing/inhibition/silences; kube-state-metrics vs metrics-server vs node-exporter (know which answers which question).
- **The golden signals + USE/RED methods** — frame every monitoring answer with these; it instantly sounds senior.
- **What to alert on for K8s itself:** apiserver latency/error rate, etcd fsync latency, pod restart storms, pending pods, node NotReady, certificate expiry. "Alert on symptoms, not causes."
- **Logs:** node-agent pattern (Fluent Bit/Vector → Loki/Elastic), why sidecar logging mostly lost, structured logging.
- **Traces:** OpenTelemetry as the standard — collector deployment patterns, sampling strategies. You need to *converse* here, not be an expert.
- **SLI/SLO/error budgets:** be ready to define an SLO for a service you ran and what you did when the budget burned.

## 5. Troubleshooting methodology (the interview centerpiece)

Have a **spoken framework**, not vibes. Mine, after 20 years:

1. **Scope:** one pod, one node, one namespace, or cluster-wide? One service or all traffic? (This halves the search space immediately.)
2. **Recent change?** deploys, node rotations, cert expiry, upgrades — 80% of incidents.
3. **Follow the request path** for traffic issues: DNS → LB → Service → endpoint list → probe status → container port → NetworkPolicy.
4. **Follow the lifecycle** for workload issues: events → describe → logs (current and `-p`) → exec → node (`journalctl -u kubelet`, `crictl ps`).
5. **Control plane last:** apiserver logs, etcd health, controller leader logs.

Memorize the differential for the big five: `Pending` (resources, taints, affinity, PVC binding, quota), `CrashLoopBackOff` (app exit, bad probe, missing config, OOM 137), `ImagePullBackOff` (name/tag, auth, registry rate limit), `NotReady` node (kubelet, CNI, disk/memory pressure, cert), `Terminating` stuck (finalizers, unreachable node, volume detach).

## 6. Multi-tenancy, quotas, and cost

- Namespace isolation stack: ResourceQuota + LimitRange + NetworkPolicy + RBAC — and its limits (soft multi-tenancy); vCluster/dedicated clusters for hard isolation.
- Cost: requests-vs-usage gap as the #1 waste source, showback with OpenCost/Kubecost, bin-packing, spot/preemptible strategies with PDBs protecting availability. FinOps questions are now standard in platform interviews.

## 7. Stateful & data operations

- Backup/DR: Velero (cluster state + volume snapshots), etcd snapshots vs app-level backups — know which protects against what (etcd snapshot ≠ your Postgres data).
- Running databases on K8s: the operator pattern answer (CloudNativePG etc.) and the honest "when I'd still use RDS" answer.

---

## Labs

1. **Build the kubeadm cluster** (Project 1 kickoff): 1 CP + 2 workers on VMs, Cilium CNI, then upgrade it one minor version with zero workload downtime.
2. **etcd fire drill:** snapshot, then destroy etcd data dir, restore from snapshot. Time yourself; do it twice.
3. **Break/fix gauntlet** (script these breaks, fix from symptoms): stop kubelet on a node; delete the CNI config; corrupt a kubeconfig cert path; fill a node's disk; set an impossible PDB then try to drain.
4. **Full observability stack:** kube-prometheus-stack + Loki; build a dashboard with the four golden signals for your Phase 1 app; fire a real alert to a webhook.
5. **Autoscale trifecta:** HPA on custom RPS metric under k6 load; KEDA scale-to-zero on a Redis queue; (cloud) Karpenter/CA node scale-up from Pending pods.
6. **conntrack/DNS lab:** reproduce DNS latency with ndots + parallel lookups, then fix with NodeLocal DNSCache and show the before/after p99.

## Interview lens

- "API server is down — what still works?" (running pods keep running; no scheduling/healing/kubectl; kubelet static pods fine — great mental-model probe)
- "Walk me through upgrading a production cluster with zero downtime. What can go wrong?" (skew, PDBs, webhook incompatibilities, deprecated APIs)
- "etcd loses 2 of 3 members — blast radius? Recovery?"
- "Node goes NotReady — timeline of what Kubernetes does, minute by minute."
- "You see p99 latency spikes every 30s on one service — go." (probe? DNS? conntrack? GC? — they're testing your *method*, narrate the framework)
- "iptables vs eBPF dataplane — why does it matter at 10k services?"

## Exit criteria

- [ ] Built, upgraded, and etcd-restored a kubeadm cluster from scratch.
- [ ] Passed the break/fix gauntlet fixing from symptoms only, under 15 min per scenario.
- [ ] Can speak the troubleshooting framework cold and apply it to any symptom an interviewer invents.
- [ ] Dashboard + alert built by hand; can define golden signals without notes.
- [ ] **This is the CKA point — sit the exam.**
