# CS 6620 Research Topic Presentation

## Serverless Computing Meets LLM Inference: Current Research and Future Directions

A ~15-minute research presentation for **CS 6620 — Cloud Computing (Spring 2026)** exploring the intersection of serverless computing and Large Language Model inference.

**Team size:** 2 people

## Topic

**Assigned Paper**: *Survey on Serverless Computing* — Hassan, Barakat & Sarhan (Journal of Cloud Computing, 2021)

**Focus**: How serverless computing is evolving to support LLM inference workloads, and how LLMs are being used to improve serverless platforms themselves.

## Papers

### Core Papers (presented in depth)

| # | Paper | Venue | Year |
|---|-------|-------|------|
| 1 | Survey on Serverless Computing (Hassan et al.) | J. Cloud Computing | 2021 |
| 2 | ServerlessLLM: Low-Latency Serverless Inference for LLMs (Fu et al.) | OSDI '24 (USENIX) | 2024 |
| 3 | LLM-Based Misconfiguration Detection for AWS Serverless Computing (Wen et al.) | ACM TOSEM | 2026 |

### Supplementary Papers (mentioned briefly)

| # | Paper | Venue | Year |
|---|-------|-------|------|
| 4 | Advancing Serverless Computing for Scalable AI Model Inference (Wang et al.) | WoSC '24 (ACM) | 2024 |
| 5 | Illuminating the Hidden Challenges of Serverless LLM Systems (Samanta & Nguyen) | WoSC '25 (ACM) | 2025 |
| 6 | Towards Resource-Efficient Serverless LLM Inference with SLINFER (Xu et al.) | HPCA '26 (IEEE) | 2026 |

## Repository Structure

```
.
├── README.md                    # This file
├── ppt.py                      # Script to generate the PowerPoint presentation
├── presentation.pptx            # Generated presentation (10 slides)
├── presentation_words.md        # Speaker notes / script (~15 min)
├── literature_review.md         # Detailed literature review of all 6 papers
├── requirements.md              # Assignment requirements
├── base.bib                     # BibTeX references for all 6 papers
├── figures/                     # Generated figures used in the presentation
│   ├── serverless_architecture.png
│   ├── research_timeline.png
│   ├── serverlessllm_architecture.png
│   ├── bidirectional_relationship.png
│   └── system_comparison.png
└── papers/                      # Source PDF papers (not tracked in git)
    └── *.pdf
```

## How to Generate the Presentation

### Prerequisites

```bash
pip install python-pptx
```

### Generate

```bash
python ppt.py
```

This creates `presentation.pptx` with 10 professionally designed slides.

### Slide Overview

| Slide | Content | Time |
|-------|---------|------|
| 1 | Title Slide | ~30s |
| 2 | Background: What is Serverless Computing? | ~90s |
| 3 | Assigned Paper: Survey (Hassan et al., 2021) — deep dive | ~3 min |
| 4 | The New Frontier: LLM Inference on Serverless | ~90s |
| 5 | ServerlessLLM: Solving Cold Start (Fu et al., OSDI 2024) | ~2.5 min |
| 6 | ServerlessLLM: Evaluation & Impact | ~2 min |
| 7 | The Reverse: LLMs Improving Serverless — SlsDetector | ~2.5 min |
| 8 | Synthesis + Open Research Question | ~2 min |
| 9 | Conclusion & Thank You | ~40s |
| 10 | References (backup) | — |
| | **Total** | **~15 min** |

### Suggested 2-Person Split

- **Person A**: Slides 1–5 (Background + Survey + ServerlessLLM intro) — ~7.5 min
- **Person B**: Slides 6–9 (ServerlessLLM eval + SlsDetector + Synthesis + Conclusion) — ~7.5 min

## Key Takeaways

1. Serverless computing is evolving from general FaaS to AI-specific platforms
2. **Cold start** is the #1 barrier — ServerlessLLM achieves 10–200× improvement
3. The relationship is **bidirectional**: serverless serves LLMs, LLMs improve serverless
4. **Open question**: Can LLM agents autonomously manage serverless infrastructure?
