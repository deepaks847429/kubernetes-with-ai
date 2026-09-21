# Labs

Working space for hands-on work. Keep each phase's lab artifacts in a subfolder here (`labs/phase1/`, `labs/phase2/`, …) so they double as portfolio evidence and as fodder for Project 3's break/fix scripts.

## Quick start (Windows)

Install tools (via [Chocolatey](https://chocolatey.org/) or [Scoop](https://scoop.sh/)):

```powershell
choco install kubernetes-cli kubernetes-helm kind k9s kubectx stern trivy   # (names vary by source)
```

Docker Desktop must be running (kind runs nodes as containers).

Spin up the default lab cluster:

```powershell
kind create cluster --name lab --config labs/kind-cluster.yaml
kubectl get nodes           # 3 nodes: 1 control-plane, 2 workers
k9s                         # live TUI dashboard
```

Tear it down and rebuild whenever — that's the point:

```powershell
kind delete cluster --name lab
```

## When kind isn't enough

- **Phase 2 (cluster lifecycle, etcd, upgrades) and Project 1** need real VMs — use Vagrant/VirtualBox or 2–3 cheap cloud VMs with `kubeadm`. You can't feel the control plane inside kind.
- **Phase 6 (GPUs)** needs a real GPU — rent a cloud spot GPU for a few hours; tear it down after.
- **NetworkPolicy labs** need a real CNI (Calico/Cilium) — kind's default doesn't enforce policy.

## Tooling cheat-set worth installing early
`kubectl`, `helm`, `kind`, `k9s` (TUI), `kubectx`/`kubens` (context/ns switch), `stern` (multi-pod logs), `kubectl-neat` (clean YAML output), `dive` (image layers), `trivy` (scanning), and later `kubebuilder`, `argocd` CLI, `cosign`, `kube-bench`.

## Discipline
- Everything as code — check configs into the repo, never click-ops your labs.
- After each session, write a war story in [../PROGRESS.md](../PROGRESS.md).
- Destroy clusters between sessions; rebuilding from config is a skill too.
