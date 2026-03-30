# AutoEIT Evaluation Test

This repository contains my solution for the AutoEIT evaluation task.

The implementation prioritizes interpretability and alignment with human scoring behavior, rather than treating the task as a purely black-box prediction problem.

---


##  Approach & Evaluation

### Approach

The core challenge of this task is inherently **multi-dimensional**: a transcription can be grammatically flawed yet semantically correct, or grammatically clean yet semantically incorrect. Therefore, a meaningful scoring system must explicitly balance **meaning preservation** and **linguistic quality**, rather than relying on a single signal.

To reflect this, I designed a **two-component scoring framework** inspired by the provided rubric:

---

#### 1. Semantic Fidelity (Meaning Preservation)

To quantify how well a response preserves the meaning of the stimulus, I used a Sentence-BERT model (`all-MiniLM-L6-v2`). Both the stimulus and response are encoded into dense vector representations, and **cosine similarity** is computed between them.

This allows the system to move beyond surface-level matching and capture **semantic equivalence**, even in the presence of paraphrasing or structural variation.

---

#### 2. Grammatical Quality (Linguistic Correctness)

To assess linguistic correctness, I used LanguageTool to detect grammatical issues in the response. The total number of detected errors serves as a proxy for fluency and correctness.

---

#### Design Insight: Decoupling Meaning and Form

A key design decision was to **treat meaning and grammar as complementary but not strictly dependent dimensions**.

This is directly aligned with the evaluation rubric:

- A response that preserves meaning but contains grammatical errors should still receive a relatively high score (e.g., 3)
- A response that fails to preserve meaning, even if grammatically correct, should be penalized more heavily

This reflects a prioritization of **semantic fidelity over surface correctness**, while still incorporating grammar as a refining signal.

---

### Data Preparation

Before feature extraction, the data was carefully cleaned to avoid introducing noise into the scoring process:

- Removed annotations (e.g., text inside parentheses and brackets)
- Ensured that only the actual linguistic content was evaluated

This step is critical, as even minor artifacts can distort similarity scores and grammar analysis.

---

### Empirical Threshold Tuning

Rather than selecting scoring thresholds arbitrarily, I followed a **data-driven tuning process**:

1. Introduced reference scores (provided sample scores) as a proxy for human evaluation
2. Computed:
   - Semantic similarity scores
   - Grammatical error counts
3. Visualized relationships using:
   - Similarity vs. score
   - Error count vs. score
   - Combined multi-dimensional scatter plots

These visualizations made it possible to observe patterns such as:
- High similarity clustering around higher scores
- Increasing grammar errors correlating with score degradation
- Overlapping regions where grammar differentiates otherwise similar responses

Based on these observations, thresholds were iteratively adjusted to better align with the rubric behavior.

---

### Scoring Function

The final scoring function combines both signals using rule-based thresholds:

- Very high similarity + minimal errors → Score 4
- High similarity with minor grammatical issues → Score 3
- Moderate similarity and/or noticeable errors → Score 2
- Low similarity or high error count → Score 1 or 0

This design explicitly encodes the rubric’s logic, particularly the idea that:
> **meaning preservation can compensate for imperfect grammar, but not vice versa**

---

### Evaluation Strategy

To evaluate the system, I compared model predictions against the reference scores using multiple complementary metrics:

- **Exact Accuracy**  
  Measures strict agreement with reference scores

- **±1 Tolerance Accuracy**  
  Accounts for subjectivity in scoring by allowing small deviations

- **Mean Absolute Error (MAE)**  
  Captures the average scoring deviation

- **Quadratic Weighted Cohen’s Kappa**  
  Measures agreement while accounting for the ordinal structure of scores (0–4)

---

### Results

**Sheet 38004-2A**
- Exact Accuracy: 60.00%
- ±1 Tolerance Accuracy: 93.33%
- MAE: 0.47
- Cohen’s Kappa: 0.71

**Sheet 38006-2A**
- Exact Accuracy: 53.33%
- ±1 Tolerance Accuracy: 96.67%
- MAE: 0.50
- Cohen’s Kappa: 0.64

---

### Interpretation

The high ±1 tolerance accuracy indicates that the model successfully captures the **relative ordering and general scoring trends**, even when exact matches are challenging.

This is expected given:
- The **subjective nature** of scoring
- The **coarse granularity** of discrete score levels
- The reliance on **proxy reference scores** rather than fully validated annotations

Cohen’s Kappa further confirms a **substantial level of agreement**, particularly considering the ordinal nature of the task.

---

### Limitations & Future Work

- The reference scores are proxies and may not perfectly reflect ground truth
- The scoring function is rule-based and manually tuned

Future improvements could include:
- Learning thresholds automatically via regression or classification
- Incorporating more nuanced linguistic features (e.g., fluency or alignment)
- Exploring more advanced embedding models for deeper semantic understanding
