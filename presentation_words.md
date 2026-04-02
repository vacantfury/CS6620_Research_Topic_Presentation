# Presentation Script: Serverless Computing Meets LLM Inference

> **Total time target: ~15 minutes (2-person team)**
> Suggested split: Person A covers slides 1–5 (~7.5 min), Person B covers slides 6–9 (~7.5 min)

---

## Slide 1: Title Slide (~30 seconds)

Good [morning/afternoon], everyone. Today we'll be presenting on "Serverless Computing Meets LLM Inference: Current Research and Future Directions." We've reviewed our assigned paper — a comprehensive survey on serverless computing — along with two additional papers that explore the rapidly growing intersection of serverless computing with Large Language Models. Let's dive in.

---

## Slide 2: What is Serverless Computing? (~90 seconds)

Let me start with the basics. Serverless computing, despite its name, doesn't mean there are no servers. It means the *developer* doesn't have to worry about them.

As you can see in this architecture diagram on the left, serverless computing has two main components. At the bottom, Backend-as-a-Service, or BaaS, provides managed cloud services — databases, storage, API gateways. On top, Function-as-a-Service, or FaaS, is where developers deploy their actual code as event-driven functions.

Let me highlight five key characteristics on the right:
- **Auto-scaling** — your application scales up and down automatically based on demand.
- **Pay-per-use** — you're only charged when your function is actually running. If it's idle, you pay nothing.
- **No server management** — the cloud provider handles provisioning, patching, scaling.
- **Event-driven** — functions are triggered by events like HTTP requests or queue messages.
- **Stateless** — and this is important for later — functions are ephemeral. They don't maintain state between invocations.

The two main models are FaaS — deploying code as functions — and BaaS — consuming managed backend services. Most people use "serverless" to mean FaaS specifically.

---

## Slide 3: Assigned Paper — Survey on Serverless Computing (~3 minutes)

Now let's look at our assigned reading. Hassan, Barakat, and Sarhan published this comprehensive survey in the Journal of Cloud Computing in 2021. This is *the* definitive survey on serverless computing — they systematically reviewed 275 research papers spanning 2016 to 2020.

On the left, you can see the scope of their work. They compared ten major FaaS platforms — from AWS Lambda and Google Cloud Functions to open-source options like Apache OpenWhisk and Knative. They identified eight application domains where serverless is being used, from chatbots and IoT to security and file processing. They also analyzed different pricing models across providers.

To put this in historical context, serverless was launched by AWS Lambda in 2014, Google and Microsoft followed in 2016, and by 2020 we had a mature ecosystem with over ten commercial platforms.

Now, on the right — and this is the most important part for our discussion — they catalog the major challenges. Performance issues appeared in 33 papers — the most common challenge — including scheduling overhead and service calling latency. Cold start latency appeared in 17 papers: that's the delay when a function needs to initialize from scratch, which can take seconds. Security concerns, vendor lock-in, limited execution duration, and the stateless design all round out the key challenges.

Here's the critical takeaway at the bottom: **this 2021 survey has no coverage of AI or machine learning workloads**. At the time, deploying ML models on serverless wasn't mainstream. But as we'll see, LLM inference has become one of the most exciting — and challenging — applications for the serverless paradigm.

---

## Slide 4: The New Frontier — LLM Inference on Serverless (~90 seconds)

So why would we want to deploy LLMs on serverless platforms? Three reasons.

First, **elastic scaling**. LLM workloads are extremely bursty — you might get zero requests for minutes, then hundreds in seconds. Serverless is built for this; it scales automatically.

Second, **cost efficiency**. GPUs are incredibly expensive. If you're running a dedicated GPU cluster and it sits idle between requests, you're burning money. With serverless, you pay only for active inference time.

Third, **operational simplicity**. Developers can focus on their application logic without managing GPU clusters, driver updates, or capacity planning.

But — and this is a big "but" — LLMs fundamentally break the assumptions that serverless computing was built on.

**Cold start is catastrophic** for LLMs. Traditional serverless cold start is milliseconds. Loading a 70-billion parameter model can take *minutes*. That's orders of magnitude worse.

**The stateless design conflicts directly** with how LLMs work. KV-cache, conversation history, multi-turn reasoning — these all require state that persists across function invocations.

**GPU provisioning** doesn't fit the serverless model. Serverless was designed for lightweight CPU functions, not allocating expensive GPUs.

These challenges motivated a wave of research, and we'll now look at the most impactful system paper addressing them.

*[Brief footnote: Wang et al. at WoSC 2024 published the first survey categorizing ML, DL, and LLM inference workloads on serverless — our reference [4] if you're interested in the taxonomy.]*

---

## Slide 5: ServerlessLLM — Solving the Cold Start Problem (~2.5 minutes)

ServerlessLLM, published at OSDI 2024 — one of the premier systems conferences — directly tackles the cold start problem.

The key insight is shown in this diagram on the left. GPU servers have a rich storage hierarchy: GPU memory at the top, DRAM, NVMe SSDs, and remote storage at the bottom. Existing model-loading tools like PyTorch and Safetensors don't exploit this hierarchy effectively — they load checkpoints through slow, generic paths.

ServerlessLLM makes three core innovations:

**First, multi-tier checkpoint loading.** They designed a new checkpoint format optimized for sequential, chunk-based reading. Combined with a multi-stage loading pipeline that saturates bandwidth at every tier, this achieves **3.6 to 8.2 times faster** checkpoint loading across models from 2.7 billion to 70 billion parameters.

**Second, LLM live migration** — and they're the first to implement this. When a request needs to move to a different server, instead of migrating the entire KV-cache — which can be gigabytes — they only migrate the tokens and re-compute the KV-cache at the destination. This dramatically reduces network traffic and enables effective load balancing.

**Third, locality-aware scheduling** through their Phantom system. It uses cost models to estimate how long it would take to load a model from each storage tier on each server, then picks the optimal one to minimize startup latency.

The result callout at the bottom speaks for itself: **10 to 200 times latency improvement** over KServe and Ray Serve, evaluated with real Azure serverless traces.

---

## Slide 6: ServerlessLLM Evaluation & Impact (~2 minutes)

Let me go a bit deeper into the evaluation, because the experimental setup is impressive and the results are convincing.

On the hardware side, they tested on two configurations: a single server with 8 A5000 GPUs, 1TB of DDR4 memory, and NVMe SSDs in RAID 0; and a cluster of 4 servers each with 4 A40 GPUs connected by 10Gbps Ethernet.

They tested across the model spectrum: OPT from 2.7B to 66B, LLaMA-2 from 7B to 70B, and Falcon 7B to 40B. They used real LLM datasets — GSM8K for math reasoning and ShareGPT for chat workloads — and generated bursty request patterns from the Azure Serverless Trace.

The three key results are striking: 3.6 to 8.2x faster checkpoint loading, 10 to 200x latency improvement over baselines, and 4.4x speedup for loading LoRA adaptors.

Why does this matter? It proves that serverless LLM inference is practical at production scale. The design is generalizable — this isn't tied to one specific GPU. And it directly addresses cold start, the #1 challenge from our survey.

In the "Related Recent Work" card at the bottom left, I want to briefly highlight two other relevant papers. SLINFER from HPCA 2026 extends this work by adding heterogeneous CPU+GPU resource sharing, achieving 86 to 154% capacity improvement. And Samanta and Nguyen at WoSC 2025 published a vision paper proposing a three-layer architecture for unified serverless LLM systems. These show this is an active and growing research area.

---

## Slide 7: The Reverse — LLMs Improving Serverless (~2.5 minutes)

Now for what I think is the most interesting paper. Wen et al., published in ACM Transactions on Software Engineering and Methodology in 2026, flips the entire narrative around. Instead of using serverless to serve LLMs, they ask: can LLMs improve serverless computing itself?

Look at the diagram on the left. So far, we've been talking about the top arrow — serverless serving LLMs, with systems like ServerlessLLM. Wen et al.'s work is the bottom arrow — LLMs improving serverless. This is a *bidirectional, symbiotic relationship*.

The problem they tackle is misconfiguration detection. AWS has over 800 resource types with incredibly complex dependency relationships. These aren't theoretical concerns: a single S3 bucket misconfiguration exposed over 50,000 scanned IDs, and DoorDash had 4.9 million customers breached from an API misconfiguration. The traditional data-driven approaches to catching these need large training datasets and can't handle the complexity.

Their system, SlsDetector, takes a completely different approach. It uses **zero-shot prompting** — no training data at all. It applies **Chain-of-Thought reasoning** so the LLM works through configurations step by step. And it defines **multi-dimensional constraints** aligned specifically to serverless configuration semantics.

The results on the right are remarkable: 72.88% precision, 88.18% recall, and a 79.75% F1-score. That's roughly **50 percentage points better** than state-of-the-art data-driven methods across all metrics. And it works across multiple LLMs — not just ChatGPT-4o, but also Llama, Gemini, and DeepSeek.

This is a fascinating result: LLMs have enough understanding of serverless configurations to detect misconfigurations that traditional methods miss.

---

## Slide 8: Synthesis & Open Research Question (~2 minutes)

Let me pull everything together.

On the left, you can see the narrative arc of our presentation. We started with the 2021 survey, which established that serverless computing is mature but cold start, statelessness, and security are unsolved. We identified the gap — no coverage of LLM workloads. ServerlessLLM from 2024 solved cold start with multi-tier loading, achieving 10 to 200x improvement. And SlsDetector showed that the relationship is actually bidirectional — LLMs can improve serverless itself.

On the right, our proposed open research question: **Can LLM agents autonomously manage serverless infrastructure?**

This extends SlsDetector's approach from *static* detection — finding misconfigurations in config files — to *real-time, autonomous* infrastructure management.

Why is this a promising research direction? First, SlsDetector already proves that LLMs understand serverless configurations. Second, auto-scaling, resource allocation, and scheduling are all decision problems that LLMs could reason about — not just detect errors, but actively optimize. Third, it would close a fascinating loop: LLMs running on serverless infrastructure, managed by LLMs — a self-optimizing system. And fourth, it sits at the intersection of cloud computing and AI, which is exactly what this research area is about.

---

## Slide 9: Conclusion & Thank You (~40 seconds)

To wrap up, four key takeaways:

1. Serverless computing is evolving from general-purpose FaaS to AI-specific platforms.
2. Cold start is the number one barrier, and ServerlessLLM achieves 10 to 200x improvement.
3. The relationship between serverless and LLMs is bidirectional — they improve each other.
4. Our open question: Can LLM agents autonomously manage serverless infrastructure?

Thank you for your attention. We're happy to take any questions.

---

## Slide 10: References (backup — not presented)

This slide lists all 6 references: 3 core papers [1]–[3] and 3 supplementary papers [4]–[6]. Only pull this up if someone asks for a specific citation.
