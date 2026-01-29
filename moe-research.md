---
layout: homepage
title: Expert Specialization in Multimodal Diffusion Models
---

# Investigation of Expert Specialization in Multimodal Diffusion Models

**Authors:** Pavel Vasilev, Daniil Tikhonov  
**Mentor:** Gleb Molodtsov  
**Project:** Yandex School of Data Analysis (YSDA)

---

## 1. Motivation & Problem Statement

Unified multimodal models often struggle because different tasks (e.g., text understanding vs. image generation) require fundamentally different "knowledge" and processing patterns. When all this knowledge is mixed in a single dense Feed-Forward Network (FFN), the model tries to satisfy all objectives simultaneously, often leading to suboptimal performance or "forgetting."

**Our Hypothesis:**  
By replacing dense FFNs with **Mixture of Experts (MoE)** layers, the model can autonomously "route" tokens to specialized experts. We hypothesized the model would separate knowledge based on:
1.  **Token Type:** Image tokens vs. Text tokens.
2.  **Data Domain:** Different underlying datasets.

## 2. Methodology

We experimented with the **Show-o** architecture (1.3B parameters), which unifies Autoregressive (text) and Masked Diffusion (image) modeling.

<div style="text-align: center; margin: 20px 0;">
    <img src="assets/img/figure_5.png" alt="Show-o Architecture" style="max-width: 100%; border-radius: 5px;">
    <p><em>Base Model: Show-o Unified Transformer (Xie et al., 2024)</em></p>
</div>

### Approaches Tested
1.  **Naive MoE:** Creating experts by simply copying existing FFN weights.
2.  **Dense-to-MoE (Smart Initialization):** Using **Taylor Scores** to identify important neurons for specific tasks and grouping them into specialized experts (Text Experts, Image Experts, Shared Experts).

### Training the Router
We used a **GShard Gate** with Top-2 selection. To encourage specialization, we employed:
*   **Load Balancing Loss:** With a decaying coefficient ($\alpha \to 0$) to allow specialization after initial training.
*   **Softmax Temperature Annealing:** Starting with high temperature for exploration and cooling down for exploitation.

## 3. Results

### A. Token Separation (Text vs. Image)
We observed a distinct separation of experts handling text and image tokens. The visualizations below show the gating probability distributions for different layers.

<div style="display: flex; justify-content: space-between; margin: 20px 0;">
    <img src="assets/img/gates/23_1.png" alt="Gate Distribution 1" style="width: 48%;">
    <img src="assets/img/gates/23_2.png" alt="Gate Distribution 2" style="width: 48%;">
</div>
<p style="text-align: center;"><em>Separation of experts for text and image tokens.</em></p>

### B. Image Generation Quality (FID)
We compared the Fréchet Inception Distance (FID) of our MoE model against the baseline.
*   **Baseline:** ~9.24
*   **MoE:** ~20

While the FID score increased (indicating lower quality), the model successfully learned to route tokens, proving the concept of autonomous specialization.

<div style="text-align: center; margin: 20px 0;">
    <img src="assets/img/fid.png" alt="FID Score" style="max-width: 80%;">
</div>

### C. Initialization with Taylor Scores (Dense-to-MoE)
We attempted to initialize experts by calculating the importance of neurons (Taylor Scores) for text and image tasks.

<div style="text-align: center; margin: 20px 0;">
    <img src="assets/img/segments.png" alt="Expert Segments" style="max-width: 80%;">
    <p><em>Extracting experts from segments based on importance scores.</em></p>
</div>

**Result:** This initialization method improved the **MMU Score** from **62% to 65%**.

## 4. Challenges & Unsuccessful Experiments

### D. Data Domain Separation (Failed)
We hypothesized that the model would separate tokens based on their source dataset (Data Domain), creating specialized "domain experts."
To encourage this, we even introduced a **Learnable Bias** ($b_i(t)$) to the router, which we gradually annealed.

**Outcome:**
Despite our efforts, the model **failed to separate tokens by data domain**. The gating distribution remained mixed across different datasets, suggesting that domain-level specialization is not as "natural" for the model as modality-level specialization (Text vs. Image).

<div style="text-align: center; margin: 20px 0;">
    <img src="assets/img/domains_exp.png" alt="Domain Separation Experiment" style="max-width: 100%;">
    <p><em>Gating distribution shows no clear separation by data domain.</em></p>
</div>

### E. Data Quality Issues
We encountered significant challenges with the quality of the training data, which likely impacted the model's ability to learn fine-grained specializations. Issues included:
*   **Poor Captioning:** Many images had irrelevant or generic captions.
*   **Artifacts:** Presence of UI elements, watermarks, or corrupted data in the dataset.

<div style="display: flex; justify-content: space-between; margin: 20px 0;">
    <img src="assets/img/xdata.png" alt="Data Issue 1" style="width: 32%;">
    <img src="assets/img/dataset_recaptioning.png" alt="Data Issue 2" style="width: 32%;">
    <img src="assets/img/mail.png" alt="Data Issue 3" style="width: 32%;">
</div>
<p style="text-align: center;"><em>Examples of data quality issues encountered during research.</em></p>

## 5. Conclusion & Future Work

We explored two approaches to scaling and specializing unified models:
1.  **Scaling (Naive MoE):** Increasing parameter count by copying FFNs. Validated on Text-to-Image (T2I).
2.  **Efficiency (Dense-to-MoE):** Maintaining parameter count by splitting FFNs. Improved Multimodal Understanding (MMU).

**Future Plans:**
*   Use **Sparse Autoencoders (SAE)** to identify semantic features activated by different tokens.
*   Construct experts based on these semantic features for better interpretability.

---
[< Back to Home](./)
