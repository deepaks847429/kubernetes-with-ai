# Phase 0 — Foundations: Linux, Networking, Containers

**Duration:** 1–2 weeks · **Why it exists:** Kubernetes is a distributed Linux system. Interviewers at serious shops probe *under* Kubernetes first — a candidate who can't explain what a container actually is fails in the first 10 minutes, regardless of how many Helm charts they've written.

---

## 1. Linux internals (the parts K8s is built on)

- Processes, signals (SIGTERM vs SIGKILL — this is literally pod termination), exit codes (137 = OOMKill, 143 = SIGTERM).
- **Namespaces** (pid, net, mnt, uts, ipc, user) and **cgroups v2** (cpu.max, memory.max, PSI). A container = namespaces + cgroups + a chroot-like root filesystem. Be able to say this in one sentence and then go three levels deeper.
- systemd basics (kubelet runs as a unit), journald, `journalctl -u kubelet`.
- Filesystems & mounts: overlayfs (how image layers become a rootfs), bind mounts (how volumes work), tmpfs (emptyDir.medium=Memory, secrets).
- CPU scheduling: CFS quota/period — **this is what CPU limits actually are**, and why "CPU throttling with low CPU usage" happens. Top-tier interview question.

**Lab:** Create a "container" by hand with `unshare --pid --net --mount --fork`, mount a rootfs, add a cgroup memory cap, OOM it. No Docker involved. If you can do this, "what is a container?" is a free win forever.

## 2. Networking (where 70% of real K8s pain lives)

- TCP/IP mechanics: handshake, states (why TIME_WAIT/CLOSE_WAIT pile up), MTU/fragmentation (VXLAN overlays lower effective MTU — classic cluster bug).
- DNS: recursion, records, TTLs, `resolv.conf` — especially **ndots and search domains**, because K8s sets `ndots:5` and it causes the most famous DNS performance issue in the ecosystem.
- Routing, NAT, and **iptables/nftables** (kube-proxy programs these), conntrack (exhaustion = mystery dropped connections under load).
- Linux virtual networking: veth pairs, bridges, network namespaces — a pod's network is exactly this.
- Load balancing concepts: L4 vs L7, health checks, connection draining.
- TLS: handshake, SANs, cert chains, expiry (cluster certs expiring is a rite of passage).

**Lab:** Two network namespaces, veth pair, bridge, NAT out to the internet with iptables. That's a hand-built pod network; CNI plugins automate this.

## 3. Containers & OCI, properly

- Image anatomy: layers, manifests, digests vs tags (why `:latest` in prod is an incident waiting to happen), multi-arch manifests.
- Registries: pull-through caches, `imagePullSecrets`, rate limits.
- **containerd** (what kubelet actually talks to via CRI), runc, and where Docker fits historically ("Docker deprecated?" — know the real answer: dockershim removal, Docker-built images still run fine because OCI).
- Dockerfile craft: multi-stage builds, non-root USER, minimal bases (distroless/alpine trade-offs), `.dockerignore`, layer caching, HEALTHCHECK vs K8s probes.
- `crictl` and `nerdctl` basics — on a broken node you won't have `docker`.

**Lab:** Take a fat 1GB+ image, get it under 100MB with multi-stage + distroless, run as non-root with a read-only rootfs. Inspect layers with `dive`.

## 4. Supporting skills

- **YAML sharp edges:** anchors, multiline strings (`|` vs `>`), the Norway problem (`no` → false), quoting numbers.
- **Git:** rebase vs merge, tags, protected branches — GitOps (Phase 4) assumes fluency.
- **Bash/scripting:** loops over kubectl output, `jq`, `awk` enough to parse logs. Plus **reading Go** (not writing yet) — K8s error messages, controller code, and Phase 5 all speak Go.
- **One cloud's primitives** (pick the one dominant in your target market — AWS globally, Azure in enterprise-heavy markets): VPC/VNet, subnets, security groups, IAM, load balancer types. EKS/GKE/AKS questions are really cloud-networking questions.

---

## Interview lens

What they actually ask at this layer:

- "What happens, at the Linux level, when a container hits its memory limit?" (cgroup OOM kill → exit 137 → kubelet restarts per restartPolicy)
- "Why is my container throttled at 30% CPU usage?" (CFS quota periods, multi-threaded bursts)
- "Explain what a container is without saying the word Docker."
- "A pod can't resolve an external hostname but IPs work — where do you look?" (CoreDNS, ndots, upstream resolvers, network policy)
- "Difference between an image and a container?" — a junior filter, answer it crisply in 15 seconds.

## Exit criteria

- [ ] Built a namespace/cgroup "container" by hand and can narrate it from memory.
- [ ] Can draw the life of a packet from one network namespace to another via veth/bridge.
- [ ] Can explain cgroups v2 CPU throttling and memory OOM semantics.
- [ ] Shrunk and hardened a real image; can defend every Dockerfile line.
- [ ] 15-second and 2-minute answers ready for "what is a container?"
