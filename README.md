## 🧠 Approach & Evaluation

### Approach

The task was to automatically score transcriptions based on how well they match a given stimulus.  
To achieve this, I modeled the problem using two complementary dimensions:

1. **Semantic Similarity**
   - I used a Sentence Transformer model (`all-MiniLM-L6-v2`) to encode both the stimulus and the response.
   - Cosine similarity was computed to measure how close the meanings are.
   - This captures whether the response conveys the same information as the stimulus.

2. **Grammatical Quality**
   - I used LanguageTool to detect grammatical errors in the response.
   - The number of detected errors serves as a proxy for linguistic correctness.

These two features reflect the core scoring criteria:
- Meaning preservation (similarity)
- Language correctness (grammar)

---

### Scoring Strategy

A rule-based scoring function was designed to combine both features into a final score (0–4).

- High similarity with zero errors → highest score
- Moderate similarity and few errors → mid scores
- Low similarity or many errors → lower scores

The thresholds were selected based on:
- Observing distributions of similarity scores and error counts
- Analyzing their relationship with reference (human) scores
- Iteratively adjusting values to better align with human judgment

---

### Evaluation

To evaluate the model, I compared the predicted scores with available reference scores using:

- **Exact Accuracy**  
  Percentage of predictions that exactly match the reference score

- **±1 Tolerance Accuracy**  
  Measures how often predictions are within ±1 of the reference  
  (important since scoring is subjective)

- **Mean Absolute Error (MAE)**  
  Average difference between predicted and reference scores

- **Cohen’s Kappa (Quadratic Weighted)**  
  Measures agreement while accounting for the ordinal nature of scores

---

### Observations

- The model performs better under ±1 tolerance than exact matching, indicating it captures general trends but struggles with fine distinctions.
- Semantic similarity is a strong signal, but grammar errors help refine scoring.
- Some disagreement arises due to the subjective nature of scoring and the absence of fully reliable annotated data.

---

### Limitations & Future Improvements

- The reference scores were approximated, which limits evaluation reliability.
- The rule-based thresholds are manually tuned and may not generalize well.

Possible improvements:
- Learn thresholds automatically using regression or classification models
- Use more advanced language models for deeper semantic understanding
- Incorporate additional features (e.g., word-level alignment or fluency metrics)
