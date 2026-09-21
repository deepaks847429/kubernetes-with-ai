# Portfolio Projects

Eight projects that double as (a) the deepest way to learn and (b) a public portfolio interviewers can inspect. **Put projects 1, 3, 4, 6 minimum in public GitHub repos** with real READMEs, architecture diagrams, and a "what I learned / what I'd do differently" section — that last section is what senior interviewers actually read.

Each project below lists: **goal · what it teaches · acceptance criteria · the interview story it earns you.** The interview story is the point — after each project you should be able to tell a 2-minute narrative that answers a real interview question with "I built that."

Do them roughly alongside the matching phase. Don't wait until you've "finished learning" — the breakage *is* the learning.

---

## Project 1 — Cluster from Scratch (the "I actually understand it" project)
**Phase:** 2 · **Goal:** Stand up a 3-node HA-ish Kubernetes cluster with kubeadm on VMs (VirtualBox/Vagrant or cloud VMs) — no managed control plane, no shortcuts.

**Teaches:** control-plane components, PKI/certs, etcd, CNI install, joining nodes, and the upgrade + etcd-restore drills. This is the single most credibility-building project you can do.

**Acceptance criteria**
- [ ] `kubeadm`-built cluster, 1 control plane + 2 workers, Cilium (or Calico) CNI.
- [ ] Deploy a real app across it; survive a worker reboot.
- [ ] Perform a minor-version upgrade (control plane then nodes) with a running workload staying available.
- [ ] Take an etcd snapshot, destroy etcd, **restore from snapshot**, prove the app/state returned.
- [ ] Document every step and every thing that broke.

**Interview story:** *"Walk me through the Kubernetes control plane / how you'd upgrade a cluster / what happens if etcd dies."* You answer from having done it, not read it.

---

## Project 2 — Production-Grade App Platform
**Phase:** 1→2 · **Goal:** Take a multi-service app (e.g. web + API + Postgres + Redis + a worker) from bare manifests to production-shaped.

**Teaches:** the full core surface applied together — probes, resources/QoS, HPA, PDB, anti-affinity/topology spread, StatefulSet for Postgres, Ingress/Gateway, NetworkPolicy, ConfigMap/Secret hygiene, zero-downtime rollouts.

**Acceptance criteria**
- [ ] Every workload has right-sized requests/limits, correct probes (incl. startup), and a PDB.
- [ ] HA spread across nodes via topologySpreadConstraints; survives a node drain with zero downtime (prove under load).
- [ ] Postgres as a StatefulSet with persistence that survives pod deletion.
- [ ] Default-deny NetworkPolicy with only the required flows opened.
- [ ] A documented rollout that drops **zero** requests (k6/hey proof), plus the same rollout deliberately breaking traffic to show you understand *why*.

**Interview story:** *"How do you run a real app in production / achieve zero-downtime deploys / lay out HA?"*

---

## Project 3 — The Break/Fix Runbook (the troubleshooting showcase)
**Phase:** 2 · **Goal:** A repo of **scripted cluster breakages** plus the runbooks that diagnose and fix them. You build the chaos *and* the cure.

**Teaches:** systematic troubleshooting — the highest-value interview skill. Also demonstrates you can write runbooks (a senior/on-call signal).

**Acceptance criteria**
- [ ] ≥10 scripted scenarios: CrashLoop, ImagePull, Pending-on-taints/quota/PVC, OOMKill, node NotReady (kubelet/CNI/disk), DNS failure, stuck Terminating (finalizer), impossible-PDB drain block, cert expiry, conntrack/DNS latency.
- [ ] Each has: a break script, the symptoms, a diagnosis walkthrough (commands + reasoning), and the fix.
- [ ] A one-page **troubleshooting framework** at the top (your spoken method from Phase 2).

**Interview story:** the entire live-troubleshooting round — and you can literally send interviewers the repo. This is the highest-ROI portfolio piece for most roles.

---

## Project 4 — GitOps Delivery Platform
**Phase:** 4 · **Goal:** A complete GitOps setup: Argo CD reconciling apps from Git, Helm-packaged, Kustomize env overlays, progressive delivery, secrets done right.

**Teaches:** how modern delivery actually works — the day-job for most K8s roles.

**Acceptance criteria**
- [ ] Argo CD (App-of-Apps or ApplicationSets) driving dev/staging/prod from a Git repo.
- [ ] App packaged as a tested Helm chart (helm unittest) with Kustomize overlays per env.
- [ ] A merge to Git deploys; a manual `kubectl edit` self-heals back.
- [ ] Argo Rollouts (or Flagger) canary that auto-promotes on good metrics and auto-rolls-back on bad.
- [ ] No plaintext secrets in Git (External Secrets Operator / sealed-secrets).
- [ ] CI pipeline that builds/scans/signs and updates the GitOps repo via PR.

**Interview story:** *"How do you deploy to production / manage GitOps secrets / do safe progressive rollouts?"*

---

## Project 5 — A Real Kubernetes Operator
**Phase:** 5 · **Goal:** Build an operator with Kubebuilder that manages a custom resource end to end.

**Teaches:** the control-loop worldview from the *inside*; the staff-level extensibility signal.

**Acceptance criteria**
- [ ] A CRD (e.g. `Website`) that provisions Deployment + Service + Ingress from a small spec.
- [ ] Full reconcile: create/update/delete with owner references (cascade delete works).
- [ ] Status subresource with conditions; demonstrates **level-triggered self-healing** (delete a managed resource → operator recreates it).
- [ ] A validating webhook rejecting bad specs; a defaulting webhook filling defaults.
- [ ] Bonus: a `v2` API with a conversion webhook.

**Interview story:** *"How would you extend Kubernetes / explain the operator pattern / level-triggered reconciliation?"* — and "here's one I wrote."

---

## Project 6 — LLM Inference Platform (the 2026 differentiator)
**Phase:** 6 · **Goal:** Serve an open LLM on Kubernetes with sane autoscaling and cost awareness.

**Teaches:** the AI-era stack almost no other candidate has hands-on. Highest salary leverage right now.

**Acceptance criteria**
- [ ] A small open model served with **vLLM** on K8s (KServe or Deployment+Service in front).
- [ ] Load-tested; **autoscaled on queue depth / KV-cache (via KEDA), not CPU**; scale-to-zero demonstrated with cold-start measured.
- [ ] GPU sharing shown (MIG or time-slicing) *or*, if no GPU, thoroughly documented + a CPU-served small model with the same autoscaling architecture.
- [ ] A GPU/inference cost+utilization dashboard (Prometheus + DCGM if GPU).
- [ ] Bonus: serve an untrusted-input path under gVisor/Kata (AI×security).

**Interview story:** *"How would you run LLM inference on Kubernetes at scale?"* — the narrative that ends interviews early in your favor.

---

## Project 7 — Secure Multi-Tenant Cluster
**Phase:** 3 · **Goal:** Harden a cluster for untrusted/multi-team workloads and prove the controls with attacks.

**Teaches:** the security depth banks/enterprise/fintech require globally; threat-model fluency.

**Acceptance criteria**
- [ ] Tenant namespaces with ResourceQuota + LimitRange + RBAC + default-deny NetworkPolicy.
- [ ] Pod Security Admission `restricted` + Kyverno policies (no privileged, signed images only, no `:latest`, limits required).
- [ ] Supply chain: Trivy scan gate + cosign sign + admission verification.
- [ ] A written threat model + an **attack demo** (container escape / IMDS SSRF / SA-token theft) shown *failing* against your controls, with a before/after.
- [ ] kube-bench remediation notes.

**Interview story:** *"Threat-model a multi-tenant cluster / how do you ensure only trusted images run / contain a pod RCE."*

---

## Project 8 — Full Observability & SRE Stack (capstone integrator)
**Phase:** 2→7 · **Goal:** Wrap one of the above platforms in production-grade observability and SRE practice.

**Teaches:** ties everything together; SLO/on-call fluency that reads as senior.

**Acceptance criteria**
- [ ] kube-prometheus-stack + Loki (+ optional OTel tracing) on a real app.
- [ ] Dashboards framed by the four golden signals; alerts on **symptoms** (SLO burn, apiserver/etcd health, restart storms, cert expiry).
- [ ] A written SLO with an error budget and a documented "what we do when it burns."
- [ ] A runbook per alert; one real alert fired end-to-end to a webhook/Slack.

**Interview story:** *"How do you monitor Kubernetes / what do you alert on / define an SLO / describe your on-call philosophy."*

---

## Portfolio presentation tips
- Each README: problem → architecture diagram → how to run it → **what broke and what I learned** → what I'd do differently at scale.
- Pin the best 3–4 repos on your GitHub profile.
- A short Loom/GIF of the money demo (self-heal reverting a change; canary auto-rollback; LLM autoscaling) is worth more than paragraphs.
- Reference these projects *by name* in interviews: "I hit exactly that — in my operator project, the reconcile..." lands far harder than abstract knowledge.
