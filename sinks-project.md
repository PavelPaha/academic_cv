---
layout: homepage
title: Attention Sinks as a Signal
---

# Attention Sinks as a Signal: A Systematic Empirical Study

**Authors:** Pavel Vasilev, Shevchenko Artem  
**Affiliation:** HSE, Yandex School of Data Analysis (YSDA)  
**Resources:** [<i class="fas fa-file-pdf"></i> Paper (Overleaf)](https://www.overleaf.com/read/sfwmtptqtwqb#0665fb) | [<i class="fab fa-github"></i> Code (GitHub)](https://github.com/PavelPaha/sinks-bruh)

---

## Abstract

Attention sinks—tokens that attract disproportionate attention mass—are used for streaming and KV-cache optimization and have been proposed as an interpretability signal. We study **sink mass** as a predictor of (in)correctness on TruthfulQA multiple-choice across multiple model families. We find strong **heterogeneity**: effect direction flips by model, predictive quality is modest (AUROC $\sim$0.44–0.61), and sink mass does **not** consistently add value beyond next-token entropy. Conclusions are sensitive to chat formatting and query-set definition. We provide a reproducible pipeline and discuss design choices and challenges encountered during the study.

---

## 1. Introduction

Transformer LMs concentrate attention on the first few tokens ("attention sinks"). This is exploited for streaming and characterized empirically. It is unclear whether **sink mass**—attention probability on the first $K$ tokens—can serve as a reliable **behavioral signal** for predicting factual errors.

We conduct a systematic study on TruthfulQA MC: we define sink mass and query set $Q$ formally, fix holdout evaluation with multiple repeats, and test seven hypotheses—distribution shift by label, predictive power, added value over entropy, head/layer localization, chat-templating and query-set sensitivity, and sink–entropy correlation. 

We report:
1.  Model-dependent direction of the sink–label effect.
2.  Modest AUROC.
3.  No consistent added value beyond entropy.
4.  Sensitivity to chat template and query set $Q$.

We conclude that sink mass is a context-sensitive measurement that must be specified and validated per setting.

## 2. Methodology

### Sink Mass Definition
For a single query position $q \in Q$, layer $\ell$, and head $h$, the **sink mass** is the total attention probability on the sink tokens (first $K$ tokens):

$$ s^{(\ell,h)}_q = \sum_{j=1}^{K} \mathbf{A}^{(\ell,h)}_{q,j} $$

We aggregate this to define the **per-example sink mass**:

$$ \bar{s} = \frac{1}{|Q| \cdot |\mathcal{L}| \cdot |\mathcal{H}|} \sum_{q \in Q} \sum_{\ell \in \mathcal{L}} \sum_{h \in \mathcal{H}} s^{(\ell,h)}_q $$

### Experimental Setup
*   **Task:** TruthfulQA multiple-choice (MC).
*   **Models:** Qwen2.5 (0.5B, 1.5B, 7B), Mistral-7B-v0.3, Mistral-Nemo, Phi-2, Pythia-2.8b, TinyLlama-1.1B.
*   **Protocol:** Sink size $K=4$. Default query set $Q$ is the *last token*. Holdout test fraction 0.30 with 5 independent repeats.

## 3. Key Results

### Predictive Power
Predictive performance is **modest but non-trivial**: raw AUROC ranges from $\sim$0.44 to $\sim$0.61 across models. Sink mass can be used as a lightweight feature, but the gain over random is model-dependent.

| Model | n | Pos. rate | Raw AUROC | Logreg AUROC | Log loss |
|-------|---|-----------|-----------|--------------|----------|
| Pythia-2.8B | 415 | 0.77 | 0.513$\pm$0.049 | 0.455$\pm$0.023 | 0.546$\pm$0.002 |
| **Qwen2.5-0.5B** | 415 | 0.74 | **0.609**$\pm$0.027 | **0.609**$\pm$0.027 | 0.573$\pm$0.001 |
| Qwen2.5-7B | 282 | 0.38 | 0.465$\pm$0.042 | 0.476$\pm$0.050 | 0.666$\pm$0.003 |
| TinyLlama-1.1B | 415 | 0.74 | 0.520$\pm$0.062 | 0.439$\pm$0.022 | 0.579$\pm$0.004 |
| Phi-2 | 415 | 0.54 | 0.474$\pm$0.037 | 0.526$\pm$0.037 | 0.691$\pm$0.007 |
| Mistral-7B-v0.3 | 415 | 0.43 | 0.442$\pm$0.028 | 0.558$\pm$0.028 | 0.681$\pm$0.005 |
| Mistral-Nemo | 415 | 0.40 | 0.450$\pm$0.045 | 0.550$\pm$0.045 | 0.676$\pm$0.007 |

### Added Value Beyond Entropy
We fit logistic regressions on entropy only (A) vs. entropy + sink mass (B). The deltas are **not** consistently positive. Aggregate $\Delta\mathrm{AUROC} \approx -0.012$. This suggests that next-token entropy often already captures the signal that sink mass might provide.

| Model | $\Delta$AUROC | $\Delta$log loss |
|-------|---------------|------------------|
| Pythia-2.8B | $-$0.0052 | $-$0.00088 |
| Qwen2.5-0.5B | +0.0090 | +0.00189 |
| Qwen2.5-7B | $-$0.0678 | $-$0.00472 |
| TinyLlama-1.1B | $-$0.0200 | $-$0.00442 |
| Phi-2 | $-$0.0070 | $-$0.00458 |
| Mistral-7B-v0.3 | +0.0069 | $-$0.00471 |
| Mistral-Nemo | $-$0.0025 | $-$0.00283 |

### Sensitivities
*   **Chat Templating:** Switching the template (auto vs. off) can **flip the sign** of the sink–label effect.
*   **Query Set:** Using a range of query positions vs. the last token materially affects AUROC.

## 4. Conclusion

Sink mass is a **context-sensitive** signal. It is affected by model family, prompt formatting, and query set definition. While it carries modest predictive signal in some configurations, it does not uniformly add value beyond next-token entropy. We recommend treating it as a measurement that must be validated per setting—not as a universal detector of hallucination.

---
[< Back to Home](./)
