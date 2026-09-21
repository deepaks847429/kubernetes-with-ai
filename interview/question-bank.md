# Question Bank

200+ questions organized by topic and seniority. **How to use:** don't just recall the answer — say it *out loud* in the format the round demands (15-sec trivia vs 2-min deep-dive). The note after hard questions tells you **what the interviewer is actually listening for** — that's usually mechanism and trade-off reasoning, not keywords.

Legend: 🟢 junior/screening · 🟡 mid · 🔴 senior/staff.

---

## Foundations (Linux / networking / containers)
1. 🟢 What is a container, without saying "Docker"? *(namespaces + cgroups + a rootfs; they want the Linux primitives)*
2. 🟢 Image vs container? Container vs VM?
3. 🟡 What actually happens when a container exceeds its memory limit? *(cgroup OOM-kill → exit 137 → kubelet restarts per policy)*
4. 🔴 A container is CPU-throttled at 30% usage — how? *(CFS quota/period; bursty multi-thread work hits quota within a period)*
5. 🟡 What are cgroups v2 and what do CPU/memory limits map to?
6. 🟡 Explain overlayfs and how image layers become a writable rootfs.
7. 🔴 Trace a packet between two network namespaces on one host. *(veth pair → bridge → routing; this is a pod network by hand)*
8. 🟡 Why does `ndots:5` cause DNS problems? What's the fix?
9. 🔴 conntrack table exhaustion — symptoms and cause under load?
10. 🟢 What is containerd's role, and did Kubernetes "remove Docker"? *(dockershim removal; OCI images still run)*
11. 🟡 Multi-stage Docker build — why, and how does it shrink images?
12. 🟡 SIGTERM vs SIGKILL and how they map to pod termination.

## Architecture & core
13. 🟢 Name the control-plane components and one job each.
14. 🔴 **Walk me through everything that happens on `kubectl apply -f deploy.yaml`.** *(the marquee question — authn/authz/admission → etcd → controllers → scheduler → kubelet → CRI/CNI/CSI → probes → endpoints → kube-proxy)*
15. 🟡 What is etcd and why an odd number of members? *(Raft quorum)*
16. 🔴 etcd loses 2 of 3 members — blast radius and recovery?
17. 🟡 What does the scheduler do — filtering vs scoring?
18. 🔴 API server is down — what still works? *(running pods run; no scheduling/healing/kubectl; static pods fine)*
19. 🟡 What's a control loop / reconciliation? Give an example.
20. 🟡 What does kube-proxy do and what's replacing it? *(iptables/IPVS → eBPF)*
21. 🟢 What are CRI, CNI, CSI? Name an implementation of each.
22. 🔴 Informers/watch caches — why don't controllers hammer the API server?

## Workloads
23. 🟢 Deployment vs StatefulSet vs DaemonSet — when each?
24. 🟡 How does a rolling update achieve zero downtime? Where can it *still* drop traffic? *(endpoint removal races termination; preStop sleep + readiness)*
25. 🔴 Explain graceful shutdown end to end. *(SIGTERM → preStop → grace period → SIGKILL; endpoints removed in parallel)*
26. 🟡 What do maxSurge/maxUnavailable do? What is progressDeadlineSeconds?
27. 🟡 StatefulSet scale-down: what happens to the PVCs, and why?
28. 🟡 Init containers vs sidecar containers (native) vs ephemeral containers.
29. 🟢 CrashLoopBackOff — top 4 causes and how you'd tell them apart.
30. 🟢 ImagePullBackOff — differential diagnosis.
31. 🟡 Job vs CronJob; what is backoffLimit, podFailurePolicy, concurrencyPolicy?
32. 🔴 Why is idempotency your problem with Jobs/CronJobs?

## Networking & services
33. 🟢 The 4 Service types and when to use each.
34. 🔴 Client pod curls a ClusterIP — trace the packet to the destination pod. *(DNS → VIP → iptables/IPVS/eBPF DNAT → pod IP → CNI → conntrack return)*
35. 🟡 The Kubernetes network model — the core guarantees?
36. 🟡 Ingress vs Gateway API — differences and why Gateway API exists.
37. 🟡 Headless service — what and why?
38. 🟡 EndpointSlices vs Endpoints — why the change?
39. 🔴 Why do per-service iptables rules degrade at 10k services, and what does eBPF change?
40. 🟡 NetworkPolicy: who enforces it? Default-deny pattern?
41. 🔴 `externalTrafficPolicy: Local` vs `Cluster` — trade-off? *(source IP + no extra hop vs even balancing)*
42. 🟡 Why might DNS be slow in your cluster, and how do you fix it? *(ndots, UDP conntrack; NodeLocal DNSCache)*
43. 🔴 Do you need a service mesh? When would you say *no*? *(they respect the "no" with reasons)*

## Config & storage
44. 🟢 ConfigMap vs Secret; is a Secret encrypted? *(base64 ≠ encryption; EncryptionConfiguration/KMS)*
45. 🟡 env var vs volume mount for config — which updates live?
46. 🟡 PV / PVC / StorageClass — how does dynamic provisioning work?
47. 🟡 Access modes: RWO vs RWX — what actually supports RWX?
48. 🔴 `WaitForFirstConsumer` — what problem does it solve? *(zone-affinity deadlock between PV and pod)*
49. 🟡 Reclaim policies — how do you rescue data with `Retain`?
50. 🟡 CSI snapshots/clones — what for?

## Scheduling & resources
51. 🟢 Requests vs limits — what does each actually control? *(requests→scheduling+CFS weight; limits→throttle/OOM)*
52. 🟡 QoS classes and eviction order.
53. 🔴 Should you set CPU limits? Argue both sides. *(throttling vs noisy-neighbor protection)*
54. 🟡 Taints/tolerations vs affinity — repel vs attract; combine for dedicated pools.
55. 🟡 topologySpreadConstraints — what problem over plain anti-affinity?
56. 🔴 How can a PodDisruptionBudget block a node drain forever?
57. 🟡 PriorityClasses and preemption — how do they interact with QoS?
58. 🟢 Pod stuck Pending — full differential. *(resources, taints, affinity, PVC, quota)*

## Health & lifecycle
59. 🟢 Liveness vs readiness vs startup probes.
60. 🔴 A production horror story for misusing liveness? For readiness? *(liveness restart loop on slow start → startup probe; readiness=liveness copy → cascading restarts under load)*
61. 🟡 Node MemoryPressure/DiskPressure — what does kubelet do?

## Cluster ops (senior core)
62. 🔴 Walk me through a zero-downtime production upgrade. What can go wrong? *(version skew, PDBs blocking drain, webhook incompat, removed APIs)*
63. 🔴 Node goes NotReady — minute-by-minute, what does Kubernetes do?
64. 🔴 How do you back up and restore a cluster? etcd snapshot vs Velero — what does each protect?
65. 🟡 The version skew policy — who can lag whom?
66. 🟡 Static pods — what, where, who manages them?
67. 🔴 Cluster certs expired — how do you even diagnose and fix it?
68. 🟡 cordon vs drain vs delete node.
69. 🔴 How does leader election work for scheduler/controller-manager?

## Autoscaling
70. 🟢 What does HPA scale on and what's the formula? *(desired = ceil(current × metric/target))*
71. 🟡 HPA + VPA on the same metric — what goes wrong, how to combine safely?
72. 🔴 Cluster Autoscaler vs Karpenter — key difference? *(node groups vs right-sized provisioning + consolidation)*
73. 🟡 What is KEDA and what can it do that HPA can't? *(event-driven, scale-to-zero)*
74. 🔴 Autoscale a service on requests-per-second, not CPU — how?

## Observability & SRE
75. 🟢 The four golden signals.
76. 🟡 metrics-server vs kube-state-metrics vs node-exporter — which answers what?
77. 🔴 What do you alert on for Kubernetes *itself*? *(symptoms: apiserver latency/errors, etcd fsync, restart storms, pending pods, NotReady, cert expiry)*
78. 🟡 "Alert on symptoms not causes" — what does that mean concretely?
79. 🟡 Write a PromQL for the 99th-percentile latency of a service. *(histogram_quantile over rate of the bucket)*
80. 🔴 Define an SLO and an error budget for a service you've run; what happens when it burns?
81. 🟡 Logging architecture on K8s — why the node-agent (not sidecar) pattern?

## Troubleshooting (expect these live)
82. 🔴 p99 latency spikes every 30s on one service — go. *(narrate the framework: probe? DNS? conntrack? GC? cron?)*
83. 🟡 Pod is Running but not serving traffic — where do you look? *(readiness, endpoints, service selector, NetworkPolicy, container port)*
84. 🟡 A pod is stuck Terminating — why? *(finalizers, unreachable node, volume detach)*
85. 🔴 Half of requests to a service fail — how do you localize it? *(one endpoint? one node? one AZ? — bisect the topology)*
86. 🟡 OOMKilled — how do you confirm and fix? *(exit 137, dmesg/events, raise limit or fix leak)*
87. 🔴 Everything's slow after a node pool rotation — hypotheses? *(cold caches, DNS, image pulls, capacity, topology spread)*

## Security
88. 🟢 The 4 C's of cloud-native security.
89. 🔴 Threat-model a multi-tenant cluster with untrusted workloads. *(the flagship — kill chain + layered controls)*
90. 🟡 How does a pod get cloud credentials without static keys? *(IRSA/Workload Identity + projected tokens)*
91. 🟡 PodSecurityPolicy is gone — what now, and its gaps? *(PSA + Kyverno/Gatekeeper)*
92. 🔴 Someone gets RCE in a pod — contain the blast radius. What did earlier decisions buy you?
93. 🟡 securityContext hardening — your default set. *(runAsNonRoot, readOnlyRootFS, drop ALL caps, seccomp RuntimeDefault, no privesc)*
94. 🔴 How do you ensure only trusted images run in prod? *(scan + sign/cosign + verify at admission)*
95. 🟡 How does a container escape happen? Name three vectors. *(privileged, hostPath /, runtime socket)*
96. 🟡 Secrets are base64 — walk from that fact to a real solution. *(KMS encryption at rest + External Secrets/Vault)*
97. 🔴 Why can `create pods` RBAC equal cluster-admin? *(mount any SA/hostPath)*
98. 🟡 What is IMDS SSRF and how do you prevent credential theft? *(egress policy + IRSA)*

## GitOps / delivery / platform
99. 🟢 What is GitOps? Push vs pull, and why pull is more secure.
100. 🔴 How do you manage secrets in GitOps? *(sealed-secrets / ESO / SOPS — never plaintext in Git)*
101. 🟡 Helm vs Kustomize — when each, why not just one?
102. 🟡 Argo self-heal and drift detection — what happens on a manual edit?
103. 🔴 Design a dev→staging→prod promotion flow with rollback.
104. 🟡 Progressive delivery: canary vs blue-green — mechanism and when.
105. 🔴 What does a platform team build, and how do you measure if the platform is good? *(self-service, golden paths, DORA/lead-time)*
106. 🟡 Where does CI end and CD begin in GitOps? *(CI → commit; CD = controller reconciles)*

## Extending Kubernetes
107. 🟡 CRD vs controller vs operator — precise definitions.
108. 🔴 How would you extend Kubernetes? Name the options, pick one for X. *(CRD/webhook/aggregated API/scheduler plugin/driver)*
109. 🔴 Level-triggered vs edge-triggered reconciliation — why it matters.
110. 🟡 Your operator dies mid-reconcile — what happens? *(idempotent resume from actual state)*
111. 🟡 When would you *not* build an operator? *(judgment signal)*
112. 🔴 How can finalizers wedge a namespace in Terminating forever?
113. 🟡 What is a mutating admission webhook? Give a real use. *(sidecar injection, defaulting)*

## AI-era Kubernetes
114. 🔴 **How would you run LLM inference on Kubernetes at scale?** *(the 3-min narrative: GPU nodes → sharing → vLLM/KServe → autoscale on queue/KV-cache → cold-start caching → cost/spot → isolation)*
115. 🟡 How do you share expensive GPUs across teams? *(MIG/time-slicing/MPS + Kueue quotas + gang scheduling)*
116. 🔴 Why can't you autoscale an LLM service on CPU? *(GPU/CPU util lies; scale on queue depth/KV-cache/TTFT)*
117. 🟡 What is gang scheduling and why do training jobs need it? *(all-or-nothing; partial = deadlock)*
118. 🔴 A training job is stuck Pending with "free" GPUs — why? *(gang admission, fragmentation, MIG profile mismatch, taints)*
119. 🟡 MIG vs time-slicing vs MPS — trade-offs.
120. 🟡 How do you handle multi-GB model cold starts? *(caching, pre-warm, scale-to-zero economics)*
121. 🔴 How would you safely run untrusted model-generated code? *(gVisor/Kata + egress policy — AI×security)*
122. 🟡 What is DRA and why does it matter for accelerators?

## System design (staff — see phases/07)
123. Design a multi-tenant platform for 500 engineers across 3 regions.
124. Design zero-downtime delivery for 200 microservices.
125. Design a GPU platform serving LLMs to multiple teams.
126. Migrate a monolith from VMs to Kubernetes — phased and risk-managed.
127. Cut a cluster's cloud bill 40% without hurting reliability.
128. Design GitOps + CI/CD for a PCI/HIPAA-regulated environment.

## Behavioral (STAR — use your own lab/work stories)
129. Tell me about a production incident you diagnosed and fixed.
130. A hard technical trade-off you made and why.
131. A time you were wrong / an outage you caused — and what changed after.
132. A disagreement on a technical decision — how it resolved.
133. Something you built that made other engineers faster.
134. How do you keep up with this ecosystem? *(point at your refresh protocol — a genuinely strong answer)*

---

### Drilling method
- 3×/week, 30 min, out loud. Rotate topics; revisit misses sooner (spaced repetition).
- For every 🔴, force yourself to state a **trade-off**, not just a fact.
- Record one deep-dive answer weekly and watch it back — you'll catch rambling and filler you can't feel live.
- Any question you can't answer → it's a gap; go back to the phase file and lab it, don't just memorize the answer here.
