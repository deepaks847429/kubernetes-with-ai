# Phase 5 — Extending Kubernetes: CRDs & Operators

**Duration:** 2 weeks · **Why it exists:** Knowing that Kubernetes is *extensible* — and being able to build the extension — is a staff/principal-level signal. Most candidates can only consume operators; being able to explain and write one puts you in a different bracket. It also finally makes the "everything is a control loop" worldview click for good.

Prereq: comfortable *reading* Go. You'll write a little.

---

## 1. The extension points (know all of them)

Interviewers ask "how would you extend Kubernetes?" — a complete answer names these and picks the right one:

- **CRDs** — new object types stored in etcd, served by the API server.
- **Custom controllers/operators** — reconcile loops acting on those (or built-in) types.
- **Admission webhooks** — mutating & validating; the interception point for policy/defaulting/injection (sidecar injection is a mutating webhook — connect this to Phase 3/4).
- **Aggregated API servers** — when a CRD isn't enough (custom storage, subresources); metrics-server is one.
- **Scheduler extensions** — scheduler plugins/framework, or a second scheduler.
- **CNI / CSI / CRI / Device Plugins / DRA** — the driver seams (device plugins & DRA matter a lot for Phase 6 GPUs).
- **Operator vs controller vs CRD** — be precise: a CRD is data; a controller is logic; an operator = CRD(s) + controller(s) encoding operational knowledge of a specific app.

## 2. The operator pattern

- **The idea:** encode a human operator's runbook (deploy, backup, failover, upgrade, scale) as software that reconciles continuously. Best explained with a stateful example: a Postgres operator that does failover and PITR backups on its own.
- **Reconciliation done right:** desired vs observed, **idempotency** (reconcile runs constantly and must be safe to re-run), **level-triggered not edge-triggered** (react to current state, not to events — the single most important operator concept, and a favorite question), status subresource & conditions, finalizers for cleanup, owner references & garbage collection, generations/observedGeneration, exponential backoff and requeue.
- **Failure thinking:** what happens mid-reconcile crash (you resume from current state — that's the point), avoiding hot loops, leader election for HA.

## 3. Building one (hands-on)

- **Tooling:** Kubebuilder / Operator SDK (controller-runtime under the hood — Manager, Client, informer-backed cache, work queues). Understand what controller-runtime hides: the informer/lister/workqueue machinery from Phase 1.
- **API design:** spec vs status, OpenAPI validation, defaulting/CEL validation rules, versioning CRDs and **conversion webhooks** (v1alpha1 → v1 migration — a real-world gotcha).
- **The Operator Maturity Model** (Levels 1–5: install → upgrades → full lifecycle → insights → autopilot) — a crisp framework to cite.
- Testing operators (envtest), and the observability of a controller (workqueue depth, reconcile latency/errors).

## 4. When *not* to build one

Senior signal = restraint. A CronJob + script or a Helm chart often beats a bespoke operator. Reach for an operator when you have genuine ongoing operational logic (stateful failover, complex lifecycle), not to deploy a stateless web app. Say this out loud in interviews.

---

## Labs

1. **Read before you write:** study a real operator's reconcile loop (e.g. CloudNativePG or cert-manager) and narrate how it handles a failure case.
2. **Build a real operator** (Project 5): Kubebuilder scaffold for a `Website` CRD that creates a Deployment + Service + Ingress from a few fields. Implement create/update/delete with owner refs so deletion cascades. Add a status with conditions.
3. **Prove level-triggered reconciliation:** manually delete the Deployment your operator created; watch it recreate it. Manually edit it; watch it revert. Kill the operator mid-reconcile; watch it resume cleanly.
4. **Add a validating webhook** that rejects invalid `Website` specs, and a defaulting/mutating one that fills defaults.
5. **Version it:** add a `v2` with a new field and a conversion webhook from `v1`.

## Interview lens

- "How would you extend Kubernetes? Walk me through the options and pick one for X." (name all extension points, justify the choice)
- "What's the difference between a CRD, a controller, and an operator?"
- "Explain level-triggered vs edge-triggered reconciliation — why does it matter?"
- "Your operator's controller pod dies halfway through creating resources. What happens?" (idempotent reconcile resumes from actual state)
- "When would you *not* build an operator?" (they're testing judgment, not knowledge)
- "How do finalizers work, and how can they wedge a namespace in Terminating forever?"

## Exit criteria

- [ ] Built, deployed, and defended a working operator with a CRD, status conditions, owner refs, and a webhook.
- [ ] Demonstrated self-healing (level-triggered) reconciliation live.
- [ ] Can name every extension point and argue which fits a given problem.
- [ ] Can articulate — with conviction — when an operator is the *wrong* tool.
