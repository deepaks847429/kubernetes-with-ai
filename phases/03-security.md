# Phase 3 — Kubernetes Security

**Duration:** 2 weeks · **Why it exists:** Mandatory at banks, fintech, healthcare, government, and any enterprise globally. Security questions also expose depth fast — you can't bluff a threat model. This phase maps to the **CKS** and to the way real security reviews are run.

Frame everything with the **4 C's**: Cloud → Cluster → Container → Code. And with a **threat-model mindset**: for each control, "what attack does this stop, and how would I get around it?"

---

## 1. The attacker's path (learn defense by learning offense)

Walk the kill chain so your defenses have a reason to exist:

1. **Get into a pod** (app RCE, SSRF, supply-chain).
2. **Container escape** (privileged pod, hostPath mount, exposed docker/containerd socket, dangerous capabilities, host namespaces).
3. **Credential theft** (the pod's ServiceAccount token, cloud instance metadata / IMDS — the SSRF-to-cloud-takeover classic).
4. **Lateral movement** (flat pod network with no NetworkPolicy, over-broad RBAC).
5. **Privilege escalation to cluster-admin** (RBAC misconfig, `escalate`/`bind` verbs, editing webhooks).
6. **Persistence** (malicious admission webhook, cronjob, DaemonSet).

Every control below breaks a specific link. In interviews, name the link.

## 2. Authentication & authorization

- authn methods: client certs, static tokens (avoid), bootstrap tokens, **OIDC** (the real enterprise SSO answer), ServiceAccount projected tokens (bound, audience-scoped, auto-rotated — vs the old forever-tokens).
- **RBAC done right:** least privilege, avoid wildcards, the dangerous verbs (`impersonate`, `escalate`, `bind`), why `create pods` can equal cluster-admin (mount any SA / hostPath), auditing RBAC with `kubectl auth can-i --list` and tools like rbac-tool/kubectl-who-can.
- Cloud IAM integration: **IRSA / Workload Identity / Azure Workload Identity** — pods assume cloud roles via projected tokens, killing long-lived cloud keys. Know at least one deeply.

## 3. Workload hardening

- **securityContext mastery:** runAsNonRoot, specific UID/GID, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, drop ALL capabilities then add back only what's needed, seccomp (`RuntimeDefault` — set it), AppArmor/SELinux basics.
- **Pod Security Admission** (the built-in replacement for the removed PodSecurityPolicy): privileged / baseline / restricted, enforced per-namespace by label; its gaps and why teams add policy engines on top.
- **Policy engines:** OPA/Gatekeeper vs Kyverno (Kyverno is the more K8s-native, easier answer; know both exist). Write policies: block privileged pods, require signed images, enforce resource limits, disallow `:latest`.
- The dangerous escapes to be able to name and block: `privileged: true`, `hostNetwork/hostPID/hostIPC`, hostPath to `/`, mounting the container runtime socket, adding `SYS_ADMIN`.

## 4. Supply chain security (huge and growing)

- Image scanning: Trivy/Grype in CI **and** admission; distinguish CVEs you must fix from noise.
- **Signing & provenance:** Sigstore/cosign, verifying signatures at admission (policy-controller/Kyverno), SLSA levels, SBOMs (Syft), attestations. This is the fastest-growing security topic in interviews post-SolarWinds/xz — have an opinion.
- Minimal/distroless/hardened base images; pinning by digest not tag.
- Admission-time enforcement: "only signed images from our registry run here."

## 5. Runtime & cluster security

- **Runtime threat detection:** Falco (eBPF/syscall rules) — detect shell-in-container, unexpected network, sensitive mounts. Know the concept even if you can't write rules fast.
- **Network security:** default-deny NetworkPolicy per namespace, egress control (block IMDS!), DNS policy; L7 policy via Cilium.
- **Secrets:** encryption at rest (KMS-backed EncryptionConfiguration, not just base64), external secret managers (External Secrets Operator + Vault/cloud KMS), why secrets in etcd/Git are a finding, sealed-secrets for GitOps.
- **API server hardening:** anonymous-auth off, audit logging on (policy tiers), admission plugin set, `--profiling` off.
- **Sandboxing** for hostile/multi-tenant workloads: gVisor, Kata Containers (VM-per-pod) — know when the extra isolation is worth the perf cost.
- **Isolation for AI/CI workloads** (ties to Phase 6): running untrusted user code / model artifacts safely.

## 6. Compliance & governance

- CIS Kubernetes Benchmark, `kube-bench` (node config) and `kube-hunter` (attack surface).
- Audit logging: what to capture, shipping to SIEM, detecting the attacker path above.
- Frameworks you'll hear named (SOC2, PCI-DSS, HIPAA, FedRAMP) — enough to map a control to a requirement.

---

## Labs

1. **Pop a cluster:** deploy a deliberately vulnerable pod (privileged + hostPath `/`), escape to the node, read another pod's SA token, use it via RBAC misconfig to escalate. Then close every hole and re-run to prove each fix.
2. **IMDS SSRF drill:** from a pod, reach cloud instance metadata; block it with egress NetworkPolicy and IRSA/Workload Identity; prove the credential theft now fails.
3. **Restricted-by-default namespace:** PSA `restricted` + default-deny NetworkPolicy + Kyverno (block privileged, require signed images, ban `:latest`, require probes/limits). Try to violate each; watch admission reject you.
4. **Signed supply chain:** build → Trivy scan (fail on criticals) → cosign sign → admission verifies signature → unsigned image is rejected.
5. **Falco:** trigger a rule with `kubectl exec ... sh`, watch the alert; tune out a false positive.
6. **kube-bench** the kubeadm cluster; remediate the top findings.

## Interview lens

- "Threat-model a multi-tenant cluster running untrusted workloads. Layer your controls." (the flagship security-design question — 4 C's + kill chain)
- "How does a pod get cloud credentials without static keys?" (IRSA/Workload Identity + projected tokens)
- "PodSecurityPolicy is gone — what now, and what are its gaps?" (PSA + Kyverno/Gatekeeper)
- "Someone gets RCE in a pod. Contain the blast radius — what did your earlier decisions buy you?"
- "How do you ensure only trusted images run in production?" (scan + sign + verify at admission)
- "Secrets are base64, not encrypted — walk me from that fact to a real solution."

## Exit criteria

- [ ] Performed and then fully remediated a container-escape + privilege-escalation attack in lab.
- [ ] Can recite the 4 C's and the kill chain, mapping each control to the link it breaks.
- [ ] Built a restricted-by-default namespace and a signed-image admission gate.
- [ ] Can hold a 20-minute threat-modeling conversation without notes.
- [ ] **This is the CKS point — sit the exam if security roles are your target.**
