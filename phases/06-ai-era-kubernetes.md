# Phase 6 — Kubernetes in the AI Era

**Duration:** 2–3 weeks · **Why it exists:** This is your **2026 unfair advantage**. Kubernetes has become the default control plane for AI/ML infrastructure, yet most candidates have *nothing* here. Even a solid working knowledge of GPU scheduling and LLM serving puts you ahead of the pack for AI-platform, ML-infra, and modern DevOps roles worldwide — the highest-paid K8s jobs right now.

> This is the **fastest-churning phase**. Re-verify every tool and version at each quarterly refresh (see README refresh protocol). Learn the *concepts* deeply; treat specific tool names as swappable.

---

## 1. GPUs on Kubernetes (the foundation)

- **Why GPUs are special:** not natively divisible like CPU, expensive, scarce — scheduling efficiency is directly money.
- **The NVIDIA GPU Operator stack:** device plugin, drivers, container toolkit, DCGM metrics, node feature discovery. What it installs and why.
- **Requesting GPUs:** `nvidia.com/gpu` resource requests, and why a GPU is (classically) all-or-nothing per container.
- **Sharing a GPU** (critical for cost, and a great interview topic): **time-slicing** (simple, no isolation), **MPS** (concurrent, light isolation), **MIG** (hardware partitioning on A100/H100-class, real isolation). Know the trade-offs cold.
- **DRA (Dynamic Resource Allocation)** — the modern replacement for the device-plugin model for flexible accelerator allocation. Check its exact GA/maturity status at refresh time; it's the future of "how K8s hands out GPUs" and a sharp thing to mention.
- Topology awareness: NUMA, NVLink/GPUDirect, why placement matters for multi-GPU training.

## 2. Batch & job scheduling for ML

The default scheduler is built for services, not training jobs. ML needs batch semantics:

- **Gang scheduling / all-or-nothing:** a distributed training job needs all N GPUs at once or none (partial = deadlock, wasted GPUs). This is *the* concept here.
- **Kueue** (the CNCF-standard job queueing layer): quotas, borrowing/preemption, fair sharing across teams. Increasingly the default answer for "how do you share a GPU cluster between teams."
- **Volcano** (the batch scheduler): gang scheduling, fair-share, the mature heavyweight.
- **Kubeflow Trainer / training operators** (PyTorchJob etc.): distributed training orchestration.
- Quota, priority, and preemption for expensive shared clusters — the FinOps-of-GPUs angle interviewers love.

## 3. LLM inference & serving (the hottest sub-topic)

- **Why serving LLMs is hard on K8s:** huge models (multi-GB weights → slow cold starts, image/weight caching), GPU memory limits, expensive idle capacity, latency SLOs, autoscaling on *the wrong metrics* (CPU/GPU util lies for LLMs).
- **The serving stack:** **vLLM** (the dominant high-throughput inference engine — PagedAttention, continuous batching; know these terms) and **KServe** / Ray Serve as the K8s serving layer (autoscaling, canary, scale-to-zero for models).
- **The right autoscaling signals:** queue depth / KV-cache utilization / time-to-first-token — via KEDA or custom metrics, **not** CPU. Connect back to Phase 2.
- **Cold-start mitigation:** model caching on nodes, pre-warming, scale-to-zero economics (KEDA), model streaming.
- **LLM-aware routing:** the emerging Gateway API **Inference Extension** (routing on model/LoRA/load) — mention it to sound current.
- **Multi-model / LoRA serving,** batching strategies, and where a GPU's KV cache becomes the bottleneck.

## 4. The broader AI-platform picture

- **Data & pipelines:** Kubeflow Pipelines / Argo Workflows for ML DAGs; feature stores; experiment tracking (MLflow) — enough to converse.
- **Ray on Kubernetes** (KubeRay): distributed Python for training/serving/tuning; when Ray vs native K8s jobs.
- **Storage for AI:** fast access to massive datasets/weights — S3 + caching, high-throughput filesystems, why `ReadWriteMany` and data locality resurface here.
- **Agentic / inference workloads** (the 2026 frontier): running LLM agents, tool sandboxes, and MCP-style services on K8s; isolating untrusted model-generated code (ties directly to Phase 3 sandboxing — gVisor/Kata). This intersection (AI + security isolation) is a standout thing to have an opinion on.
- **Cost & efficiency:** GPU utilization as the headline metric, bin-packing GPUs, spot GPUs with checkpointing, right-sizing — the GPU-FinOps story.

## 5. The narrative to walk in with

Be able to give a 3-minute answer to *"How would you run LLM inference on Kubernetes at scale?"*: GPU nodes (Operator) → sharing strategy (MIG/time-slicing by workload) → serving layer (vLLM behind KServe) → autoscale on queue depth/KV-cache with KEDA, scale-to-zero for idle models → weight caching for cold starts → cost via bin-packing + spot with checkpointing → isolation for untrusted inputs. That single coherent story will out-differentiate 95% of candidates.

---

## Labs

*(Some need a real GPU. If you have none: use a cloud spot GPU for a few hours, or do the CPU-based structural labs and study the GPU parts thoroughly — you can still speak to them.)*

1. **GPU (or simulated) scheduling:** install the NVIDIA GPU Operator on a cloud GPU node; run a CUDA pod; enable time-slicing and run two pods on one GPU; observe with DCGM/`nvidia-smi`. *(No GPU? Use a fake-device-plugin or study + narrate.)*
2. **Serve an LLM** (Project 6): deploy a small open model with vLLM on K8s; put KServe or a plain Deployment+Service in front; load-test it; **autoscale on queue depth with KEDA**, not CPU; demonstrate scale-to-zero and measure cold-start pain.
3. **Gang scheduling:** install Kueue; submit more GPU jobs than capacity; show queueing, quota borrowing, and all-or-nothing admission (no partial-GPU deadlock).
4. **GPU cost dashboard:** Prometheus + DCGM exporter → a Grafana panel of GPU utilization and $/token-ish; identify idle waste.
5. **Untrusted inference isolation:** run a model-serving or code-exec pod under gVisor/Kata; verify the extra isolation; connect to the Phase 3 threat model.

## Interview lens

- "How would you run LLM inference on Kubernetes at scale?" (the 3-minute narrative above)
- "How do you share expensive GPUs across teams and jobs?" (MIG/time-slicing/MPS + Kueue quotas + gang scheduling)
- "Why can't you just use HPA on CPU to autoscale an LLM service?" (GPU/CPU util lies; scale on queue depth / KV-cache / TTFT)
- "What is gang scheduling and why do training jobs need it?"
- "A training job is stuck Pending on a GPU cluster with 'free' GPUs — why?" (all-or-nothing gang admission, fragmentation, taints, MIG profile mismatch)
- "How do you handle multi-GB model cold starts?" (caching, pre-warm, scale-to-zero economics)
- "How would you safely run untrusted model-generated code in your cluster?" (sandboxing + egress policy — the AI×security cross)

## Exit criteria

- [ ] Served a real model on K8s and autoscaled it on a sane (non-CPU) signal.
- [ ] Can explain GPU sharing (time-slicing/MPS/MIG) and gang scheduling with trade-offs.
- [ ] Can deliver the 3-minute "LLM inference at scale" narrative cold.
- [ ] Have a defensible opinion on isolating untrusted AI workloads.
- [ ] Re-verified every tool/version in this file against current reality (it will have moved).
