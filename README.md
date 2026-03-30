# AutoEIT Evaluation Test

This repository contains my solution for the AutoEIT evaluation task.

The task: Implement a reproducible script that applies the meaning-based rubric to the sentence transcriptions (comparing learner utterances to prompt sentences provided) and outputs sentence-level scores for each utterance in the sample data files. Scores should be documented in 

The implementation prioritizes interpretability and alignment with human scoring behavior, rather than treating the task as a purely black-box prediction problem.

---


##  Approach & Evaluation

### Approach
According to the rubric attached [Spanish EIT Scoring Rubric.docx](https://github.com/user-attachments/files/26338648/Spanish.EIT.Scoring.Rubric.docx), The core challenge of this task is inherently **multi-dimensional**: a transcription can be grammatically flawed yet semantically correct, or grammatically clean yet semantically incorrect. Therefore, a meaningful scoring system must explicitly balance **meaning preservation** and **linguistic quality** with bias towards the meaning since it is a meaning based evaluation, rather than relying on a single signal.

To reflect this, I designed a **two-component scoring framework** inspired by the provided rubric:

---

#### 1. Semantic Fidelity (Meaning Preservation)

To quantify how well a response preserves the meaning of the stimulus, I used a Sentence-BERT model (`all-MiniLM-L6-v2`). Both the stimulus and response are encoded into dense vector representations, and **cosine similarity** is computed between them.

and i chose this model because it allows the system to move beyond surface-level matching and capture **semantic equivalence**, even in the presence of paraphrasing or structural variation.

---

#### 2. Grammatical Quality (Linguistic Correctness)

To assess linguistic correctness, I used LanguageTool to detect grammatical issues in the response. The total number of detected errors serves as a proxy for fluency and correctness.

---

#### Design Insight: Decoupling Meaning and Form

A key design decision was to **treat meaning and grammar as complementary but not strictly dependent dimensions**.

This is directly aligned with the evaluation rubric:

-- A response that contains grammatical errors but preserves meaning should still receive a relatively high score (e.g., 3)
-- A response that contains grammatical errors affecting the meaning should receive a lower score (e.g., 2)
- A response that fails to preserve meaning, even if grammatically correct, should be penalize

This reflects a prioritization of **semantic fidelity over surface correctness**, while still incorporating grammar as a refining signal.

---

### Data Preparation

Before feature extraction, the data was carefully cleaned to avoid introducing noise into the scoring process:

- Removed annotations (e.g., text inside parentheses and unintended texts inside brackets e.g.[cough])
- Ensured that only the actual linguistic content was evaluated

This step is critical, as even minor artifacts can affect similarity scores and grammar analysis.

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

The final scoring function combines both signals using rule-based thresholds

This design intended to encode the rubric’s logic, particularly the idea that:
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

### Limitations & Future Work

- The reference scores are proxies and may not perfectly reflect ground truth
- The scoring function is rule-based and manually tuned

Future improvements could include:
- Learning thresholds automatically via regression or classification
- Incorporating more nuanced linguistic features (e.g., fluency or alignment)
- Exploring more advanced embedding models for deeper semantic understanding
