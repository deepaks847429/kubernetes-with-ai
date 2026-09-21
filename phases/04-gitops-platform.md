# Phase 4 — GitOps, Helm & Platform Engineering

**Duration:** 2–3 weeks · **Why it exists:** This is what "the job" actually is at most companies now. Nobody runs `kubectl apply` to prod by hand anymore. Platform-engineering and "internal developer platform" roles are the fastest-growing K8s job category globally, and they hinge on this phase.

---

## 1. Packaging: Helm and the alternatives

- **Helm properly:** template mechanics (values, `_helpers.tpl`, named templates, `range`/`if`/`with`, the whitespace war), chart dependencies/subcharts, `helm upgrade --install`, release history and rollback, hooks (pre/post-install), `helm template` for debugging, `--atomic`/`--wait`. Know Helm's real weakness: it's a text templater, so it has no idea about live cluster state.
- **The alternatives and when they win:** Kustomize (overlays, no templating, `kubectl -k` native — great for environment variants), Helm+Kustomize together, and the config-language camp (Jsonnet, cue, Pkl). Have a one-line opinion on each.
- **Chart quality:** values schema validation, `helm lint`, `helm unittest`, publishing to an OCI registry.

## 2. GitOps — the core model

- **The principle:** Git is the single source of truth; a controller continuously reconciles the cluster to match Git. Push (CI runs kubectl) vs **Pull** (in-cluster agent syncs) — and why pull is more secure (no cluster creds in CI, drift auto-corrected).
- **Argo CD** (the market-share leader, and what most interviews mean by GitOps): Applications, sync waves & hooks, sync policies (auto-sync, self-heal, prune), health assessment, the App-of-Apps pattern, ApplicationSets for multi-cluster/multi-env fan-out. Run it.
- **Flux** (the other graduated option): source/kustomize/helm controllers, image-automation. Know the difference so you can speak to either shop's stack.
- **Drift detection & self-healing** — the killer GitOps demo: change something with `kubectl edit`, watch the controller revert it.
- **Secrets in GitOps** (the classic hard problem): sealed-secrets, External Secrets Operator, SOPS — you cannot commit plaintext secrets, so how? Have an answer ready.
- **Progressive delivery:** Argo Rollouts / Flagger — canary and blue-green with automated analysis (promote or roll back on metrics). This is where GitOps meets SRE.

## 3. Environment & release strategy

- Promotion across dev → staging → prod (folders/branches/overlays — and the trade-offs; the "branch-per-env is an anti-pattern" debate).
- Multi-cluster fleet management (ApplicationSets, cluster registration, hub-spoke).
- Deployment strategies end to end: rolling, blue/green, canary, shadow — mechanism *and* when each fits.

## 4. CI/CD end to end

- The pipeline: build → test → scan (Trivy) → sign (cosign) → push → **update the GitOps repo** (CI's job ends at a commit/PR; CD is the GitOps controller's job — this separation is the whole point).
- Pipeline tools you'll hear: GitHub Actions/GitLab CI (most common), and K8s-native ones (Tekton, Argo Workflows).
- Ephemeral preview environments per PR (a platform-team favorite interview topic).

## 5. Platform engineering & the developer experience

- **The thesis:** platform teams build a paved road so app teams self-serve without becoming K8s experts. "Kubernetes as an implementation detail the app dev shouldn't have to see."
- **Internal Developer Platforms:** Backstage (service catalog, scaffolding, the CNCF-standard portal), the golden-path idea.
- **Infrastructure as software:** Crossplane (provision cloud infra *through* the K8s API/GitOps — DBs, buckets, the lot) vs Terraform/OpenTofu/Pulumi (know where each fits; the "Terraform for infra, Argo for apps, Crossplane blurring the line" landscape).
- **The platform-as-product mindset:** app teams are your customers; golden paths, self-service, and paved roads over gatekeeping. Interviewers for these roles listen hard for this framing.

---

## Labs

1. **Full GitOps loop** (Project 4 kickoff): Argo CD watching a Git repo; app defined as Helm chart with Kustomize env overlays for dev/staging/prod via ApplicationSets. Change Git → cluster converges. `kubectl edit` → self-heal reverts it.
2. **Write a production-grade Helm chart** for your Phase 1 app: values schema, helpers, health/limits/HPA templated, a pre-upgrade hook, `helm unittest` coverage. Publish to an OCI registry.
3. **Progressive delivery:** Argo Rollouts canary that auto-promotes on good Prometheus metrics and auto-rolls-back on a bad version. Ship a deliberately broken version and watch it abort.
4. **GitOps secrets:** wire External Secrets Operator (or sealed-secrets) so no plaintext secret is ever in Git; rotate one and watch it propagate.
5. **CI→CD split:** a GitHub Actions pipeline that builds/scans/signs and then commits a new image digest to the GitOps repo via PR; Argo takes it from there.
6. (Stretch) **Crossplane:** provision a cloud bucket/DB via a K8s manifest through GitOps.

## Interview lens

- "What is GitOps and why is pull-based more secure than push?"
- "How do you manage secrets in GitOps when you can't commit them?"
- "Helm vs Kustomize — when do you reach for each, and why not just one?"
- "Design a promotion pipeline dev→staging→prod. How does a change reach prod, and how do you roll back?"
- "Argo detects drift — what does self-heal do, and when would you *not* want it on?"
- "What does a platform team build, and how do you measure if the platform is good?" (developer self-service, lead time, golden-path adoption — DORA metrics)

## Exit criteria

- [ ] A live GitOps repo where a merge deploys and a manual cluster edit self-heals.
- [ ] Authored, tested, and published a real Helm chart you can defend line by line.
- [ ] Ran an automated canary that promoted and one that auto-rolled-back on metrics.
- [ ] Can articulate the CI-vs-CD boundary and the platform-as-product philosophy convincingly.
