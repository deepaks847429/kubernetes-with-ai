# Kubernetes Interview Mastery Curriculum

> Built like a 20-year DevOps veteran would train their own replacement.
> Goal: walk into **any** Kubernetes interview on the planet — product company, bank, hyperscaler, AI startup — and be the strongest candidate in the room.

**Last refreshed:** 2026-09-21 · **Baseline K8s version:** verify current stable (was ~v1.34+ at last refresh — run the [Refresh Protocol](#the-dynamic-part-refresh-protocol) below)

---

## Who this is for

Someone who wants to go from "I can use kubectl" to "I can design, break, fix, secure, extend, and explain Kubernetes under pressure." Interviews at senior level are not about memorizing commands — they test whether you've **operated clusters in anger**. Every phase here forces that.

## The one rule

**Never learn a topic without breaking it.** For every concept: deploy it, break it deliberately, fix it from symptoms only, then write a 5-line war story in [PROGRESS.md](PROGRESS.md). Your war stories become your behavioral interview answers. This is the single highest-leverage habit in this whole curriculum.

---

## Roadmap at a glance

| Phase | Module | Duration | Interview weight |
|-------|--------|----------|-----------------|
| 0 | [Foundations: Linux, networking, containers](phases/00-foundations.md) | 1–2 wks | Filters out 50% of candidates |
| 1 | [Core Kubernetes](phases/01-core-kubernetes.md) | 3–4 wks | Every interview, every level |
| 2 | [Advanced operations & troubleshooting](phases/02-advanced-operations.md) | 3–4 wks | The senior-level differentiator |
| 3 | [Security](phases/03-security.md) | 2 wks | Mandatory at banks/fintech/enterprise |
| 4 | [GitOps, Helm & platform engineering](phases/04-gitops-platform.md) | 2–3 wks | What day-2 jobs actually are |
| 5 | [Extending Kubernetes (CRDs & operators)](phases/05-extending-kubernetes.md) | 2 wks | Staff-level signal |
| 6 | [Kubernetes in the AI era](phases/06-ai-era-kubernetes.md) | 2–3 wks | The 2026 differentiator — most candidates have nothing here |
| 7 | [Interview mastery & system design](phases/07-interview-mastery.md) | ongoing | Converts knowledge into offers |

**Full track:** ~16–20 weeks at 10–12 hrs/week.
**Fast track (already experienced):** 8 weeks — skim 0–1, go deep on 2, 3, 6, 7, and do Projects 3, 4, 6.

## Portfolio projects

Eight portfolio-grade assignments live in [projects/README.md](projects/README.md). Each has acceptance criteria and an "interview story" you should be able to tell afterwards. Projects 1, 3, 4, 6 are the minimum credible portfolio; put them in public GitHub repos with real READMEs — interviewers do look.

## Interview drill material

- [interview/question-bank.md](interview/question-bank.md) — 200+ questions organized by topic and seniority, with the *reasoning* interviewers listen for.
- [interview/scenarios.md](interview/scenarios.md) — live troubleshooting drills ("the cluster is broken, you have 20 minutes") that mirror CKA-style and panel-style practicals.

## Certifications (milestones, not goals)

Certs open doors globally (especially EU/Middle East/APAC recruiters filter on them); they don't get you hired alone.

- **CKA** — take after Phase 2. The practical exam format is itself great interview training.
- **CKAD** — optional if you're app-dev leaning; mostly a subset of CKA plus speed.
- **CKS** — take after Phase 3. Rare enough to be a genuine differentiator.
- **KCNA/KCSA** — skip unless an employer asks; too shallow to signal at senior level.

## Lab environment

You need clusters you can destroy daily. On your Windows machine:

1. **kind** (Kubernetes-in-Docker) — default lab; multi-node, fast to rebuild, config-as-code. Most labs assume kind.
2. **k3s / k3d** — for edge/lightweight scenarios and multi-cluster drills.
3. **kubeadm on VMs** (VirtualBox/Vagrant, or 2–3 cheap cloud VMs) — mandatory for Phase 2 cluster-lifecycle and Project 1. You cannot answer "walk me through a control-plane upgrade" credibly if you've only used managed clusters.
4. **One managed cluster** — a free/cheap EKS *or* GKE *or* AKS spell (spot/preemptible nodes, tear down after each session). Interviews worldwide assume you know at least one cloud's flavor; know one deeply and the others' differences on paper.

Tooling to install day one: `kubectl`, `helm`, `kind`, `k9s`, `kubectx/kubens`, `stern`, `kubectl-neat`, `dive`, `trivy`.

---

## The dynamic part: refresh protocol

Kubernetes ships 3 releases/year and the AI-infra layer churns faster. This syllabus stays current because **you** re-verify it on a schedule. Run this checklist **every quarter** (put a recurring reminder in your calendar now):

1. **K8s releases** — read the release notes/blog for any version shipped since last refresh. Update the baseline version at the top of this file. Specifically hunt for: features that went GA, APIs deprecated/removed (interviewers love "what changed in the last two releases?").
2. **Deprecation sweep** — run `kubectl api-resources` and a tool like `pluto` or `kubent` against your lab manifests; fix anything deprecated.
3. **CNCF landscape shifts** — check graduated/incubating project changes in the areas of this syllabus (GitOps, observability, security, AI/batch scheduling). If a tool in these files got archived or superseded, edit the file — this is a living document.
4. **AI-era module (Phase 6)** — this churns fastest. Re-check the current state of: DRA (Dynamic Resource Allocation), Kueue, KServe/vLLM serving stacks, and whatever the current "run LLMs on K8s" reference architecture is.
5. **Job-market signal** — skim 10 current Kubernetes/platform job posts (your target geography). Any skill appearing in 5+ posts that isn't in this syllabus → add it as a new section and note it in the changelog below.
6. **Re-do one timed drill** from [interview/scenarios.md](interview/scenarios.md) cold, to catch skill rot.

### Changelog

| Date | Change |
|------|--------|
| 2026-09-21 | Initial curriculum created. |

---

## How to use this repo

1. Read this file fully. Set up the lab environment.
2. Work phases in order (or fast-track). Each phase file ends with **exit criteria** — do not move on until you pass them honestly.
3. Log every lab, breakage, and war story in [PROGRESS.md](PROGRESS.md).
4. Interleave: from Phase 1 onward, do 30 minutes of question-bank drilling per week from [interview/question-bank.md](interview/question-bank.md) — spaced repetition beats cramming.
5. When you land interviews, spend the final week in Phase 7 exclusively.
