# Literature Review: Serverless Computing and LLM Inference

## Overview

This literature review examines six papers spanning the evolution of serverless computing from its foundational principles to its emerging role as an infrastructure for Large Language Model (LLM) inference. The papers collectively trace an arc from a comprehensive survey of serverless computing fundamentals (Hassan et al., 2021) to cutting-edge systems addressing the unique challenges of deploying LLMs on serverless platforms (2024–2026). The overarching theme is the tension between serverless computing's promise of elastic scalability, pay-per-use pricing, and operational simplicity versus the practical challenges introduced by the resource-intensive, stateful, and latency-sensitive nature of modern AI workloads.

---

## 1. Serverless Computing: A Survey of the Current Landscape

**Hassan, Barakat, and Sarhan (2021)** — *Journal of Cloud Computing* [Assigned Paper]

Hassan et al. provide a comprehensive survey of serverless cloud computing, systematically reviewing 275 research papers published between 2016 and 2020. The paper serves as the foundational reference for understanding the serverless paradigm.

### Key Contributions
- **Taxonomy of Serverless Computing**: The survey establishes a clear taxonomy distinguishing Backend-as-a-Service (BaaS) and Function-as-a-Service (FaaS), with FaaS identified as the dominant serverless model. Serverless computing, first introduced by AWS Lambda in 2014 and adopted by Google and Microsoft in 2016, adds an abstraction layer that frees developers from server-side management.
- **Platform Comparison**: The survey compares ten FaaS platforms (AWS Lambda, Apache OpenWhisk, Microsoft Azure Functions, Google Cloud Functions, OpenLambda, IBM Cloud Functions, OpenFaaS, Knative, Function Stage Huawei Cloud, and Nuclio), evaluating their supported languages, trigger mechanisms, pricing models, and deployment capabilities.
- **Use Cases**: Eight serverless application domains are identified: chatbots, information retrieval, file processing, smart grids, security, networks, and mobile/IoT applications.
- **Challenges Identified**: The survey catalogs key challenges including:
  - **Cold start latency** (17 papers): The delay incurred when initializing a serverless function for the first time or after idle periods.
  - **Performance** (33 papers): Scheduling overhead, service calling overhead, and resource allocation inefficiencies.
  - **Security** (13 papers): Isolation concerns from shared platform execution and trust issues with sensitive data processing.
  - **Vendor lock-in**: Strong dependence on a provider's ecosystem for data storage, transfer, and processing.
  - **Limited execution duration**: Functions are restricted to short execution times, creating barriers for long-running tasks.
  - **Resource sharing**: Efficiently sharing resources among functions remains technically challenging.

### Significance
This survey establishes the baseline understanding of serverless computing's maturity and its remaining gaps. Notably, the 2021 survey does not address AI/ML workloads in depth, highlighting a significant gap that subsequent papers in this review address.

---

## 2. Advancing Serverless Computing for Scalable AI Model Inference: Challenges and Opportunities

**Wang, Jiang, and Mi (2024)** — *WoSC '24 (ACM)*

Wang et al. provide the first comprehensive survey specifically focused on AI model inference within serverless environments, reviewing 31 high-quality papers published between 2019 and 2024 from top venues (ASPLOS, ATC, OSDI, SC).

### Key Contributions
- **AI Inference Taxonomy**: The survey categorizes AI inference workloads into three tiers—ML-based, DL-based, and LLM-based inference—and maps them against ten critical performance dimensions: resource management, cost-effectiveness, distributed inference, cold start latency, GPU utilization, bursty workloads, scheduling, batching, auto-scaling, and model partitioning.
- **Serverless AI Inference Workflow**: Users submit inference requests as stateless functions that load pre-trained models and execute inference within Docker containers. The serverless platform handles automatic scaling in response to fluctuating workloads.
- **Challenge Analysis**:
  - **Bursty workloads**: Sudden, unpredictable traffic spikes cause cold start delays and resource under-provisioning, making it difficult to balance responsiveness with cost-efficiency.
  - **Cold start latency**: Particularly pronounced for DL/LLM inference where large models must be loaded into memory, deeply degrading performance for real-time applications.
  - **Resource under/over-provisioning**: Achieving optimal balance between latency, throughput, and cost remains a persistent challenge, especially with GPU-intensive LLM workloads.
  - **Stateful workflows**: The inherently stateless serverless paradigm conflicts with models requiring complex inter-model communication or state preservation.
- **Key Research Directions**: The survey identifies energy-efficient serverless inference, AI-driven auto-scaling strategies, and advanced scheduling as promising areas for future research.

### Significance
This paper bridges the gap between the general serverless survey by Hassan et al. and the specific systems-level solutions in subsequent papers. It provides the research landscape for understanding where the field stands and what problems remain unsolved.

---

## 3. ServerlessLLM: Low-Latency Serverless Inference for Large Language Models

**Fu, Xue, Huang, Brabete, Ustiugov, Patel, and Mai (2024)** — *OSDI '24 (USENIX)*

ServerlessLLM is a landmark systems paper that proposes a complete framework for low-latency LLM inference on serverless platforms. Published at OSDI, one of the premier systems conferences, it directly addresses the cold start and model loading bottlenecks identified by the survey papers.

### Key Contributions
- **Fast Multi-Tier Checkpoint Loading**: ServerlessLLM introduces a loading-optimized checkpoint format that supports sequential, chunk-based reading and efficient tensor in-memory addressing. It exploits the multi-tier storage hierarchy of GPU servers (GPUs, DRAM, SSDs, remote storage) through:
  - An in-memory data chunk pool
  - Memory-copy efficient data paths
  - A multi-stage data loading pipeline
  - Result: **3.6–8.2× faster checkpoint loading** compared to existing systems (Safetensors, PyTorch) for models including OPT (2.7B–66B), LLaMA-2 (7B–70B), and Falcon (7B–40B).
- **Efficient LLM Live Migration**: ServerlessLLM is the first to implement LLM live migration in serverless inference systems. Instead of migrating the large KV-cache, only tokens are migrated, with efficient re-computation of the KV-cache at the destination server, significantly reducing network traffic.
- **Startup-Time-Optimized Model Scheduling (Phantom)**: A locality-aware scheduling algorithm that integrates cost models for estimating checkpoint loading times across different storage tiers and migration costs, selecting the optimal server to minimize startup latency.

### Evaluation Highlights
- Evaluated on GPU clusters with A5000 and A40 GPUs using real-world serverless workloads (Azure Trace) and LLM datasets (GSM8K, ShareGPT).
- Demonstrated **10–200× improvement in latency** compared to KServe, Ray Serve, and Ray Serve with local caching for OPT model inference.
- Also supports LoRA adaptors with **4.4× speedup** in checkpoint loading.

### Significance
ServerlessLLM provides concrete, production-ready solutions to the cold start problem that both surveys identified as a critical barrier. Its multi-tier storage design and live migration capabilities represent key advances in making serverless LLM inference practical.

---

## 4. Illuminating the Hidden Challenges of Serverless LLM Systems

**Samanta and Nguyen (2025)** — *WoSC '25 (ACM)*

This paper provides a comprehensive vision paper for efficient serverless LLM systems, performing a systematic gap analysis of existing infrastructure and proposing a three-layer architecture.

### Key Contributions
- **Gap Analysis (Table 1)**: The paper evaluates existing systems across 11 requirement dimensions (intent interface, temporality, multi-modal support, cost control, dynamic model selection, cross-task sharing, context awareness, cold start, GPU provisioning, stateful execution, SLO guarantees). Key findings:
  - Traditional serving frameworks (TensorFlow Serving, TorchServe, Triton) lack intent interfaces, cost controls, and serverless-native features.
  - Serverless platforms (AWS Lambda, Azure Functions, Google Cloud Functions) have broad coverage but lack specialized LLM optimizations for cold start, GPU provisioning, and stateful execution.
  - Specialized LLM systems (vLLM, TGI, ServerlessLLM) address specific challenges but leave significant gaps in declarative interfaces, multi-modal support, and end-to-end SLO guarantees.
- **Three-Layer Architecture**:
  1. **Declarative Interface**: Allows users to specify NLP tasks through intent-oriented queries without requiring expertise in model architecture.
  2. **Adaptive Orchestration Engine**: Automatically synthesizes and optimizes inference pipelines with dynamic model selection, cross-tenant sharing, and adaptive reconfiguration.
  3. **Efficient Serverless Runtime**: Provides computational infrastructure with abstractions for GPU allocation, stateful execution across ephemeral invocations, and KV-cache management.
- **Key Requirements Identified**:
  - **Stateful execution**: Managing conversation history, KV-cache persistence, and long-running task progress beyond single function invocations.
  - **Fine-grained GPU provisioning**: Sub-second provisioning and de-provisioning through advanced virtualization or time-slicing, with strong performance isolation.
  - **Cross-tenant resource sharing**: Model sharing (single loaded instance serving multiple users) and computational sharing (batching across users) for economic viability.

### Significance
This paper highlights that the field has been solving point problems (cold start, checkpoint loading) without a holistic architectural vision. It provides a roadmap that connects the fundamental serverless challenges (Hassan et al.) with the specific LLM inference challenges (Wang et al.) and identifies what remains beyond what systems like ServerlessLLM address.

---

## 5. LLM-Based Misconfiguration Detection for AWS Serverless Computing

**Wen, Chen, Zhu, Sarro, Liu, Ping, and Wang (2025/2026)** — *ACM Transactions on Software Engineering and Methodology*

This paper takes a complementary approach: instead of using serverless to serve LLMs, it uses LLMs to improve serverless computing itself. SlsDetector is the first framework to leverage LLMs for static misconfiguration detection in serverless applications.

### Key Contributions
- **Problem Statement**: Serverless application misconfigurations on AWS SAM (the most widely adopted configuration schema) have led to severe real-world incidents, including exposure of 50,000+ scanned IDs due to S3 bucket misconfigurations and a breach affecting 4.9 million customers from API misconfigurations.
- **SlsDetector Framework**:
  - Uses zero-shot prompting with advanced prompt engineering (no training data required).
  - Designs multi-dimensional constraints aligned with serverless configuration characteristics.
  - Leverages Chain-of-Thought (CoT) reasoning to enhance LLM inference accuracy.
  - Generates structured, deterministic responses with clear explanations.
- **Evaluation Results**: On a curated dataset of 110 configuration files (correct configurations, real-world misconfigurations, and injected errors):
  - **Precision: 72.88%**, **Recall: 88.18%**, **F1-score: 79.75%** using ChatGPT-4o.
  - Outperforms state-of-the-art data-driven methods by **53.82** (precision), **17.40** (recall), and **49.72** (F1) percentage points.
  - Demonstrates generalization across multiple LLMs: Llama 3.1 (405B), Gemini 1.5 Pro, and DeepSeek V3.

### Significance
This paper demonstrates a bidirectional relationship between serverless computing and LLMs: not only can serverless platforms serve LLMs, but LLMs can also be leveraged to improve serverless platform reliability. It addresses the configuration complexity challenge (800+ AWS resource types with complex dependency relationships) identified in the foundational survey.

---

## 6. Towards Resource-Efficient Serverless LLM Inference with SLINFER

**Xu, Li, Chen, Zhao, Tang, and Guo (2026)** — *HPCA '26 (IEEE)*

SLINFER represents the latest advancement in serverless LLM inference, published at the premier computer architecture conference. It addresses the critical problem of resource efficiency by introducing elastic sharing of heterogeneous resources (CPU + GPU).

### Key Contributions
- **Heterogeneous Resource Investigation**: SLINFER systematically investigates serving LLMs on both CPUs and GPUs, identifying sharing opportunities between them. Key insight: modern CPUs with matrix acceleration units (e.g., Intel AMX) can deliver matrix multiplication throughput comparable to GPUs for certain inference tasks.
- **Elastic Resource Sharing**: Unlike prior work (ServerlessLLM, Medusa, ParaServe) that allocates dedicated GPUs to each LLM, SLINFER implements transparent, on-demand resource sharing across models with a unified hardware abstraction.
- **Guidelines for Serverless LLM Sharing**: The paper constructs deployment guidelines that account for the unique characteristics of LLM inference (two-phase prefill/decode, fluctuating compute/memory demands) and serverless workloads (highly dynamic, bursty patterns).
- **Two Subsystem Design**:
  - A GPU sharing subsystem for elastic allocation across concurrent LLM instances.
  - A CPU-assisted subsystem leveraging AMX-enabled CPUs for offloading portions of inference.

### Evaluation Highlights
- Tested on 4 A100-80GB GPU nodes and 4 32-core Intel Xeon CPU nodes with real-world workloads (Azure Serverless Trace, Azure LLM Inference Dataset) and models (Llama-3.2-3B, Llama-2-7B, Llama-2-13B).
- **47–62% improvement in serving capacity** through elastic GPU sharing alone.
- **86–154% improvement** when additionally leveraging CPU resources.

### Significance
SLINFER pushes the boundary beyond the cold start problem (solved by ServerlessLLM) to the resource efficiency problem. It demonstrates that the future of serverless LLM inference lies in heterogeneous computing, moving beyond GPU-only solutions.

---

## Synthesis and Thematic Analysis

### Theme 1: The Cold Start Problem as the Central Challenge

The cold start problem is the thread connecting all six papers. Hassan et al. (2021) identified it as one of the top challenges of serverless computing. Wang et al. (2024) demonstrated that cold start is even more severe for AI/LLM workloads due to large model loading requirements. ServerlessLLM (2024) directly solved this with multi-tier checkpoint loading (3.6–8.2× speedup), and SLINFER (2026) further addressed it through elastic resource pre-allocation.

### Theme 2: From Stateless Functions to Stateful AI Inference

A fundamental tension exists between serverless computing's stateless design and the stateful nature of LLM inference (KV-cache, conversation history, multi-turn reasoning). Samanta and Nguyen (2025) identified stateful execution as a critical gap, while ServerlessLLM's live migration mechanism (migrating tokens rather than KV-cache) represents a pragmatic solution.

### Theme 3: The Bidirectional Relationship Between Serverless and LLMs

The papers collectively reveal a symbiotic relationship:
- **Serverless → LLMs**: Serverless platforms provide elastic, cost-efficient infrastructure for LLM inference (ServerlessLLM, SLINFER, Wang et al.).
- **LLMs → Serverless**: LLMs improve serverless platform reliability through misconfiguration detection (Wen et al.).
This bidirectional relationship suggests a co-evolutionary trajectory where advances in one domain catalyze improvements in the other.

### Theme 4: Resource Efficiency and Heterogeneous Computing

The progression from dedicated GPU allocation (pre-2024) to elastic GPU sharing (SLINFER, 2026) to CPU+GPU heterogeneous computing represents a clear maturation trajectory. SLINFER's finding that modern CPUs with matrix acceleration units can complement GPUs suggests that future serverless LLM systems will need to be hardware-agnostic and capable of dynamically leveraging whatever hardware is available.

---

## Open Research Questions

Based on this literature review, the following research questions remain open and warrant future investigation:

1. **Unified Serverless LLM Architecture**: How can the three-layer architecture proposed by Samanta and Nguyen (2025) be realized in practice, integrating the point solutions from ServerlessLLM and SLINFER into a cohesive system with end-to-end SLO guarantees?

2. **LLM-Driven Serverless Optimization**: Extending Wen et al.'s work on misconfiguration detection, can LLM agents autonomously manage serverless infrastructure—performing auto-scaling decisions, resource allocation, and performance optimization in real-time?

3. **Cost-Performance Pareto Optimization**: As heterogeneous resources become available, how should serverless platforms dynamically select between CPU, GPU, and specialized accelerators to achieve optimal cost-performance trade-offs for varying LLM workloads?

4. **Security and Privacy in Multi-Tenant LLM Serving**: The cross-tenant sharing identified as essential by Samanta and Nguyen introduces new attack surfaces. How can strong isolation guarantees be maintained while enabling the resource sharing necessary for economic viability?

---

## References

1. Hassan, H. B., Barakat, S. A., & Sarhan, Q. I. (2021). Survey on serverless computing. *Journal of Cloud Computing*, 10(39), 1–29.

2. Wang, L., Jiang, Y., & Mi, N. (2024). Advancing serverless computing for scalable AI model inference: Challenges and opportunities. *Proceedings of the 10th International Workshop on Serverless Computing (WoSC '24)*, pp. 1–6. ACM.

3. Fu, Y., Xue, L., Huang, Y., Brabete, A.-O., Ustiugov, D., Patel, Y., & Mai, L. (2024). ServerlessLLM: Low-latency serverless inference for large language models. *18th USENIX Symposium on Operating Systems Design and Implementation (OSDI '24)*, pp. 135–153.

4. Samanta, A. & Nguyen, T. G. (2025). Illuminating the hidden challenges of serverless LLM systems. *Proceedings of the 11th International Workshop on Serverless Computing (WoSC '25)*, pp. 51–57. ACM.

5. Wen, J., Chen, Z., Zhu, Z., Sarro, F., Liu, Y., Ping, H., & Wang, S. (2026). LLM-based misconfiguration detection for AWS serverless computing. *ACM Transactions on Software Engineering and Methodology*, 35(4), Article 110, 1–28.

6. Xu, C., Li, Z., Chen, Q., Zhao, H., Tang, X., & Guo, M. (2026). Towards resource-efficient serverless LLM inference with SLINFER. *IEEE International Symposium on High Performance Computer Architecture (HPCA '26)*, pp. 1–18.
