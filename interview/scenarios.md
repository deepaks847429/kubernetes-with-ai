# Live Troubleshooting Scenarios

Timed, hands-on drills that mirror CKA-style practicals and panel "the cluster is broken, fix it" rounds. **Rules:** set a timer, narrate your reasoning aloud the whole time (interviewers grade *method*), and fix from symptoms — resist jumping to the answer you half-remember.

Each scenario: **the break · the symptom you're shown · the winning approach · the trap.** Build the break scripts into Project 3. Answers are deliberately terse — the value is in *you* working it, not reading it.

Run one cold per week. If you solve it in <60% of the time budget, raise the difficulty (chain two breaks).

---

## Tier 1 — single-object (budget 10 min each)

### S1 — CrashLoopBackOff
**Break:** app exits 1 on a missing env var (ConfigMap key typo).
**Symptom:** pod restarts climbing, `CrashLoopBackOff`.
**Approach:** `describe` (events) → `logs` and `logs -p` (previous) → spot the fatal log → trace to ConfigMap/env → fix → confirm Ready.
**Trap:** assuming it's the image. Read the *app's own logs* first.

### S2 — ImagePullBackOff
**Break:** wrong tag / private registry with no imagePullSecret.
**Symptom:** `ImagePullBackOff`, container never starts.
**Approach:** `describe` → the pull error is explicit (not found vs unauthorized vs rate-limited) → fix tag or add `imagePullSecret`.
**Trap:** confusing "manifest unknown" (bad name/tag) with "unauthorized" (creds) — they read differently, fix differently.

### S3 — Pending pod
**Break:** requests exceed any node / a taint with no toleration / unbound PVC / quota exceeded.
**Symptom:** pod stuck `Pending`.
**Approach:** `describe pod` → the scheduler's message names the reason → walk the differential (resources → taints/affinity → PVC → quota).
**Trap:** looking at logs (there are none — it never scheduled). The signal is in **events**.

### S4 — OOMKilled
**Break:** memory limit far below the app's working set.
**Symptom:** restarts, last state `OOMKilled`, exit 137.
**Approach:** `describe` (last state) → confirm 137 → `kubectl top` / metrics → raise limit or fix the leak; explain requests-vs-limits.
**Trap:** just bumping the limit without asking *why* (a leak will re-OOM at any limit).

### S5 — Service returns nothing
**Break:** Service selector doesn't match pod labels (or wrong targetPort).
**Symptom:** pods Running & Ready, but curling the Service times out / connection refused.
**Approach:** `get endpoints`/`endpointslices` → **empty** → selector vs pod labels mismatch, or port mapping wrong → fix.
**Trap:** blaming the network. Empty endpoints = a selector/label problem, full stop.

---

## Tier 2 — node & cluster (budget 15 min each)

### S6 — Node NotReady
**Break:** stop kubelet, or break the CNI config, or fill the disk.
**Symptom:** node `NotReady`; its pods eventually evicted.
**Approach:** `describe node` (conditions) → SSH → `systemctl status kubelet` + `journalctl -u kubelet` → CNI? disk? cert? → fix root cause.
**Trap:** deleting/rebooting the node reflexively before reading *why* it went NotReady.

### S7 — DNS resolution failure
**Break:** scale CoreDNS to 0, or a NetworkPolicy blocks port 53, or break the CoreDNS ConfigMap.
**Symptom:** pods can't resolve names; IPs work.
**Approach:** test with a debug pod (`nslookup`) → check CoreDNS pods/logs → check the `kube-dns` Service & endpoints → check NetworkPolicy on 53 → fix.
**Trap:** editing `/etc/resolv.conf` in a pod — it's injected; the fix is upstream (CoreDNS or policy).

### S8 — Stuck Terminating
**Break:** a resource with a finalizer whose controller is gone (or a namespace with stuck contents).
**Symptom:** object/namespace stuck `Terminating` forever.
**Approach:** `get -o yaml` → spot the `finalizers` → identify the missing controller → remove the finalizer *only* once you understand what it was for.
**Trap:** force-deleting everything as a habit — in real life that finalizer may be protecting cleanup (e.g. a load balancer, a volume).

### S9 — Drain won't complete
**Break:** a PodDisruptionBudget with minAvailable equal to replica count.
**Symptom:** `kubectl drain` hangs on "cannot evict … would violate PDB".
**Approach:** read the drain error → find the PDB → realize the app can't tolerate any disruption at this replica count → scale up or fix the PDB, then drain.
**Trap:** `--force`/`--disable-eviction` to power through — that defeats the very safety the PDB exists for; explain the *right* fix.

### S10 — Rollout stuck / bad deploy
**Break:** new image fails readiness; or maxUnavailable=0 with a failing probe.
**Symptom:** `rollout status` never completes; old + new pods coexist.
**Approach:** `rollout status` → `describe`/`get rs` → new pods not Ready → logs → decide: fix forward or `rollout undo`.
**Trap:** deleting pods hoping it self-corrects — understand progressDeadlineSeconds and just roll back if it's a bad build.

---

## Tier 3 — control plane & subtle (budget 20 min each)

### S11 — etcd restore
**Break:** corrupt/lose etcd data.
**Symptom:** API server won't come up / cluster state gone.
**Approach:** `etcdctl snapshot restore` to a new data dir → point the etcd static pod manifest at it → verify API server → confirm state returned. (Rehearse; don't improvise this live.)
**Trap:** not knowing that etcd runs as a **static pod** (edit the manifest in `/etc/kubernetes/manifests`), and confusing an etcd snapshot with app data backups.

### S12 — Expired certificates
**Break:** advance clock / expire cluster certs.
**Symptom:** kubectl fails with x509 errors; components can't talk.
**Approach:** `kubeadm certs check-expiration` → `kubeadm certs renew` → restart control-plane static pods → refresh admin kubeconfig.
**Trap:** panicking. It's routine once you know the two commands — which is exactly why they ask it.

### S13 — Intermittent p99 latency spikes
**Break:** a CronJob every minute hammering a shared node, or CPU throttling from tight limits, or DNS/conntrack pressure.
**Symptom:** periodic p99 spikes, most requests fine.
**Approach:** narrate the framework — correlate spikes with cron schedule / GC / DNS; `kubectl top`, check throttling metrics, check for noisy neighbors and topology → hypothesis → test one at a time.
**Trap:** claiming a single cause immediately. This scenario grades *structured hypothesis testing*, not a lucky guess.

### S14 — Half the traffic fails
**Break:** one of three endpoints unhealthy but still selected (bad readiness), or one node's CNI degraded, or one AZ.
**Symptom:** ~33% of requests fail/timeout; the rest are fine.
**Approach:** the pattern *is* the clue — bisect the topology (which pod? which node? which AZ?); check per-endpoint health and readiness gating.
**Trap:** treating it as a global outage. "Some but not all" almost always means one bad member of a set — find the set.

### S15 — Can't schedule despite free capacity (bonus, AI-era)
**Break:** GPU nodes with free GPUs but a gang/quota (Kueue) or MIG-profile mismatch blocks admission.
**Symptom:** GPU job `Pending`, `nvidia-smi` shows idle GPUs.
**Approach:** check gang/all-or-nothing admission, Kueue quotas/borrowing, MIG profile vs request, taints/tolerations on GPU nodes.
**Trap:** assuming "free GPU = schedulable." Batch admission and accelerator profiles break that intuition — exactly why it's a great senior question.

---

## Scoring yourself
- **Did you narrate throughout?** Silence loses the round even if you fix it.
- **Did you find root cause or just make the symptom disappear?** Interviewers probe: "why did that happen?"
- **Did you use events/describe before logs/exec, in the right order?**
- **Time:** Tier 1 < 10 min, Tier 2 < 15, Tier 3 < 20. If over, note which step cost you and drill it.

### Level-up
Once each is easy solo, have someone (or a script) chain two breaks (e.g. NotReady node *and* a bad PDB) without telling you how many — that's what a real "broken cluster" round feels like.
