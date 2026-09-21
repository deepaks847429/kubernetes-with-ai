# Progress & War-Story Log

Your training journal. Two jobs: (1) track where you are, (2) **capture war stories** — every breakage, diagnosis, and "oh *that's* why" becomes a STAR behavioral answer later. Interviewers can't tell a memorized fact from lived experience *except* when you tell a specific story. This file is where those come from. Log religiously.

---

## Phase tracker

| Phase | Status | Started | Finished | Exit criteria passed? |
|-------|--------|---------|----------|----------------------|
| 0 — Foundations | ⬜ not started | | | |
| 1 — Core Kubernetes | ⬜ | | | |
| 2 — Advanced Ops | ⬜ | | | |
| 3 — Security | ⬜ | | | |
| 4 — GitOps/Platform | ⬜ | | | |
| 5 — Extending K8s | ⬜ | | | |
| 6 — AI-era K8s | ⬜ | | | |
| 7 — Interview Mastery | ⬜ (ongoing) | | | |

Status legend: ⬜ not started · 🟨 in progress · ✅ done

## Project tracker

| Project | Repo URL | Status | Money-demo recorded? |
|---------|----------|--------|---------------------|
| 1 — Cluster from Scratch | | ⬜ | |
| 2 — Prod App Platform | | ⬜ | |
| 3 — Break/Fix Runbook | | ⬜ | |
| 4 — GitOps Platform | | ⬜ | |
| 5 — Operator | | ⬜ | |
| 6 — LLM Inference | | ⬜ | |
| 7 — Secure Multi-Tenant | | ⬜ | |
| 8 — Observability/SRE | | ⬜ | |

## Certification tracker

| Cert | Target date | Status |
|------|-------------|--------|
| CKA | after Phase 2 | ⬜ |
| CKS | after Phase 3 | ⬜ |
| CKAD (optional) | | ⬜ |

---

## War-story log

> Template — copy per incident. Aim for one every lab session. Keep them **specific** (real numbers, real commands, real dead-ends). Write the STAR line last — that's the interview-ready version.

### [DATE] — [one-line title]
- **Context:** what I was doing / which lab or phase.
- **Symptom:** exactly what I saw (error text, `kubectl` output).
- **Investigation:** what I checked, in order — including the wrong turns (wrong turns make the story real).
- **Root cause:** the actual why.
- **Fix:** what resolved it.
- **Lesson / how I'd prevent it:** the takeaway.
- **STAR one-liner (interview-ready):** "When [situation], I needed to [task], so I [action], which [result]."

---

### Example (delete once you have your own)
### 2026-09-21 — Drain hung forever on a PDB
- **Context:** Phase 2 break/fix gauntlet, draining a node for a simulated upgrade.
- **Symptom:** `kubectl drain node2` hung: "cannot evict pod ... would violate the pod's disruption budget."
- **Investigation:** Checked the drain output (named the PDB) → `kubectl get pdb` showed minAvailable=3 with exactly 3 replicas → so *zero* disruption was allowed.
- **Root cause:** PDB left no room; with replicas == minAvailable, no pod can ever be evicted.
- **Fix:** Scaled the deployment to 4 (or relaxed the PDB to minAvailable=2), then drain completed.
- **Lesson:** PDBs protect availability but a too-strict one is a self-inflicted outage during maintenance. I now sanity-check PDB vs replica count before any planned drain.
- **STAR one-liner:** "When a node drain hung during a cluster upgrade, I traced it to a PodDisruptionBudget that allowed zero disruption, adjusted capacity to give it headroom, and completed the upgrade with no downtime — and added a pre-upgrade PDB check to our runbook."

---

## Weekly interview-prep log
Track the Phase 7 cadence so it doesn't slip.

| Week | Q-bank drill (3×/wk) | Timed scenario (cold) | System-design (out loud) | Notes / weak spots |
|------|----------------------|-----------------------|--------------------------|--------------------|
| | | | | |
