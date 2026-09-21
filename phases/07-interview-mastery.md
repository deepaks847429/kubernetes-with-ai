# Phase 7 — Interview Mastery & System Design

**Duration:** ongoing; go exclusive in the final week before interviews · **Why it exists:** Knowledge doesn't get you hired — *converting knowledge into offers under pressure* does. This phase is the meta-skill: how K8s interviews are actually run globally, and how to win each round.

---

## The five interview formats you'll face (worldwide)

1. **Screening / trivia** (recruiter or junior eng): rapid-fire "what is X." Win by being crisp and *never rambling*. 15-second answers.
2. **Deep-dive technical** (senior eng): they pick one thing you said and go five levels down. Win by having genuine mechanism knowledge — this is what Phases 0–2 built. You cannot bluff here.
3. **Live troubleshooting / practical** (CKA-style or a broken cluster): shared screen, "fix this." Win with your **spoken troubleshooting framework** (Phase 2) — narrate your reasoning; they're grading method, not luck. See [interview/scenarios.md](../interview/scenarios.md).
4. **System design** (staff-level, see below): "design a multi-tenant platform for 500 engineers." Win with structure and trade-offs.
5. **Behavioral / war stories:** "tell me about a production incident." Win with your PROGRESS.md war stories in **STAR** format (Situation, Task, Action, Result). This is why you logged every breakage.

## System design for Kubernetes/platform roles

A repeatable structure for any "design X on Kubernetes" prompt:

1. **Clarify & scope:** scale (pods/nodes/clusters/regions), tenancy, SLOs, compliance, team size, budget. Never start drawing before this — asking good questions *is* part of the score.
2. **Cluster topology:** how many clusters and why (blast radius, compliance, region); managed vs self-managed; node pool strategy.
3. **Multi-tenancy model:** namespaces + quotas/RBAC/NetworkPolicy (soft) vs vCluster vs dedicated clusters (hard) — pick and justify.
4. **Networking:** CNI choice, ingress/Gateway, service mesh yes/no (and *why no* is a valid, strong answer), DNS, multi-cluster connectivity.
5. **Delivery:** GitOps (Argo/Flux), promotion flow, progressive delivery, secrets management.
6. **Observability:** metrics/logs/traces stack, SLOs, on-call/alerting philosophy (symptoms not causes).
7. **Security:** the 4 C's, admission policy, supply chain, runtime detection, isolation.
8. **Scaling & cost:** autoscaling (pods + nodes, Karpenter/KEDA), FinOps, capacity planning.
9. **Reliability/DR:** failure domains, backup/restore (etcd + Velero), RTO/RPO, multi-region strategy.
10. **Day-2 & DX:** upgrades, platform-as-product, golden paths, who's on call for what.

Then state **trade-offs explicitly** ("I'd start with soft multi-tenancy because X, and move to vCluster if Y") — senior interviewers hire for judgment and trade-off reasoning, not for reciting the "right" stack.

### Design prompts to rehearse (out loud, whiteboard, 30–45 min each)
- Multi-tenant platform for 500 engineers across 3 regions.
- Zero-downtime deployment system for 200 microservices.
- CI/CD + GitOps for a regulated (PCI/HIPAA) environment.
- GPU platform serving LLMs to multiple product teams (pulls Phase 6 — a hot 2026 prompt).
- Migrate a monolith from VMs to Kubernetes (phased, risk-managed).
- Multi-region active-active with failover.
- Cut a cluster's cloud bill 40% without hurting reliability.

## Behavioral bank — prep these in STAR from your own labs/work
- A production incident you diagnosed and fixed (use a real Phase 2 gauntlet story).
- A time you made a hard trade-off (managed vs self-managed, mesh vs no mesh).
- A time you were wrong / an outage you caused and what you changed.
- Disagreeing with a team on a technical decision.
- Something you built that made other engineers faster (platform mindset).
- How you keep up with a fast-moving ecosystem (point at this repo's refresh protocol — a genuinely great answer).

## The "questions to ask them" list (you're interviewing them too, and it scores you)
On-call load & incident culture; how upgrades are handled; managed vs self-managed and why; tech-debt appetite; how platform success is measured; biggest current reliability pain. Thoughtful questions here read as senior.

## Delivery mechanics
- **Think out loud, always.** Silent problem-solving reads as being stuck. Narrate the framework.
- **Structure before detail:** headline the approach, then drill. ("Three things could cause this; I'll check them fastest-first.")
- **Admit unknowns well:** "I haven't run X in prod, but here's how I'd reason about it / find out." Far stronger than bluffing — and bluffing gets caught instantly in the deep-dive round.
- **Time-box practicals:** get *something* working, then improve. Partial credit is real.

## Study cadence
- **Spaced repetition** on [interview/question-bank.md](../interview/question-bank.md): 30 min, 3×/week, all the way through — not a night-before cram.
- **One timed scenario** from [interview/scenarios.md](../interview/scenarios.md) per week, cold, narrating aloud (record yourself; watching it back is brutal and effective).
- **One system-design prompt** out loud per week.
- **Refresh PROGRESS.md war stories** monthly so they stay interview-ready.

## Final-week checklist before a specific interview
- [ ] Research their stack (job post, eng blog, GitHub) → predict their favorite topics and bias your prep.
- [ ] Re-run one CKA-style scenario cold.
- [ ] Rehearse the `kubectl apply` walkthrough and the "LLM inference at scale" narrative.
- [ ] 3 war stories loaded in STAR.
- [ ] 5 questions ready to ask them.
- [ ] Lab environment warm (a kind cluster up) in case of a live exercise.

## Exit criteria (you're interview-ready)
- [ ] Can run your troubleshooting framework aloud on any invented symptom.
- [ ] Delivered 3+ system-design prompts out loud with clear trade-offs.
- [ ] 5+ STAR war stories from your own labs/work.
- [ ] Question-bank drilled to the point that trivia is reflexive.
- [ ] Zero bluffing reflex — you can say "I don't know, here's how I'd find out" cleanly.
