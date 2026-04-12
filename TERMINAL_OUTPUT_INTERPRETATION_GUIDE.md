# Terminal Output Interpretation Guide

This guide is meant to be read side-by-side with terminal output from:

python newsgroups_end_to_end_practice.py

It explains what each section means, the math behind it, why it matters, and what to compare.

## How to use this guide during a run

1. Run the script.
2. Pause at each major terminal header.
3. Match that header to this document.
4. Read: Purpose -> Math -> Interpretation -> Tuning -> Comparison.

---

## 1) 20 NEWSGROUPS: DATA ORIGIN AND COLLECTION CONTEXT

### Purpose
Understand where data comes from and what assumptions/risks exist.

### Why it matters
- Dataset source can introduce bias.
- Historical internet data has noisy language and topic leakage risk.

### What to compare
- With metadata removed (`headers`, `footers`, `quotes`) vs not removed.
- Performance changes can indicate leakage from formatting artifacts.

---

## 2) PREPROCESSING GUIDE

### Purpose
Show exactly how raw text is transformed before vectorization.

### Key settings explained
- lowercase: normalize case variants.
- remove_urls / remove_emails: reduce noisy tokens.
- remove_numbers: keep/remove numeric tokens.
- keep_only_letters: strips punctuation/symbols.
- min_token_length: removes short tokens below threshold.

### Why this matters mathematically
Preprocessing changes the feature space itself.
If vocabulary has size V, each document vector is x in R^V.
Changing cleaning changes V and the coordinates of x.

### Example
If min_token_length=2:
- token "i" (length 1) is dropped
- token "ai" (length 2) is kept

### What to compare
- `min_token_length=2` vs `3`
- `remove_numbers=False` vs `True`
- Observe effects on macro F1 and retrieval precision.

---

## 3) STEP 1: LOAD DATA / CACHE MESSAGES

### Purpose
Show whether data was loaded from source or from local cache.

### Expected messages
- "Loaded official 20 Newsgroups dataset..."
- or "Loaded prepared dataset from local cache."

### Why it matters
Caching speeds iteration while debugging/tuning.
Data/cleaning are expensive; model comparisons are faster if reused.

### What to compare
- First run time vs second run time with same cleaning config.

---

## 4) STEP 2: CLEAN TEXT + BEFORE/AFTER EXAMPLES

### Purpose
Validate cleaning behavior qualitatively.

### What students should inspect
- Are key topic words preserved?
- Are only noisy artifacts removed?
- Did we accidentally remove useful information (numbers, acronyms)?

### Why this matters
A model cannot recover information removed by preprocessing.

---

## 5) ROUTE 1: TEXT CLASSIFICATION (BoW + Logistic Regression)

### When to use
Use when you have labels and need predictions for new documents.

### Example task
Predict newsgroup topic; spam vs not spam; sentiment class.

### Math architecture
1. Vectorization (BoW): x in R^V
2. Class score:
   z_c = w_c^T x + b_c
3. Multinomial probability:
   p(y=c|x) = exp(z_c) / sum_j exp(z_j)
4. Training objective:
   L = -sum_i log p(y_i|x_i) + lambda ||W||^2_2
5. Prediction: argmax_c z_c

### Terminal metrics
- Accuracy: global fraction correct.
- Macro F1: average F1 across classes (equal class weight).

### Interpretation
- If accuracy is high but macro F1 lower, some classes may be weak.
- Use per-class precision/recall to identify confusion patterns.

### Tuning knobs
- ngram_range
- min_df
- max_features
- C (regularization inverse)

### Compare against
- Baseline route models
- Different C grids (0.01, 0.1, 1, 10, 100)

---

## 6) ROUTE 2: DOCUMENT CLUSTERING (TF-IDF + KMeans)

### When to use
Use when labels are unavailable and you want natural grouping.

### Example task
Group support tickets into themes.

### Math architecture
1. TF-IDF vectorization
2. KMeans objective:
   J = sum_i ||x_i - mu_{z_i}||^2
3. Alternating updates:
- Assignment: choose nearest centroid
- Update: recompute centroid means

### Terminal metrics
- Silhouette: internal separation/compactness metric.
- Homogeneity: label-alignment quality (for analysis only).

### Interpretation
- Clustering scores are usually lower than supervised classification metrics.
- Silhouette and homogeneity answer different questions.

### Tuning knobs
- k (clusters)
- TF-IDF features
- min_df and max_features
- initialization / n_init

### Compare against
- k around expected class count (for example 10, 20, 30)

---

## 7) ROUTE 3: SEARCH AND RETRIEVAL (TF-IDF + COSINE)

### When to use
Use when goal is ranking similar documents for a query.

### Example task
Find top-5 related documents.

### Math architecture
- Similarity score:
  cosine(q,d) = (q dot d) / (||q|| ||d||)
- Rank by descending cosine score.

### Terminal metric
- Precision@k: fraction of top-k retrieved items matching query label.

### Interpretation
- Per-query precision can vary a lot.
- Inspect query walkthroughs to understand failure modes.

### Tuning knobs
- k
- vectorizer type (BoW vs TF-IDF)
- query/doc preprocessing consistency
- vocabulary controls

### Compare against
- Precision@1, @5, @10 trends.
- Same model with/without aggressive cleaning.

---

## 8) ROUTE 4: BASELINE MODEL COMPARISON

### When to use
Use early to select a strong, simple benchmark before deep models.

### Why important
A strong classical baseline is often hard to beat and easier to deploy.

### Terminal output
Each pipeline prints accuracy and macro F1.

### Interpretation
- Best baseline by accuracy is your practical starting point.
- If scores are close, prioritize simpler/faster models.

### What to compare
- Count + NB vs TF-IDF + LinearSVC
- Accuracy and macro F1 together, not one metric only.

---

## 9) FINAL CROSS-ROUTE SUMMARY

### Purpose
Summarize route outcomes in one table.

### Important warning
These metrics are not directly interchangeable:
- Classification accuracy
- Clustering silhouette
- Retrieval Precision@k

Each metric should be interpreted in its own task context.

---

## Core concept comparisons students should know

### Macro F1 vs binary F1
- Binary F1 often means F1 for positive class.
- Macro F1 averages F1 across classes equally.
- In binary tasks, macro F1 is average of class 0 and class 1 F1.

### Overfitting vs underfitting signs
- Overfitting: training strong, validation/test weaker.
- Underfitting: both training and validation weak.

### Regularization C
- Larger C -> weaker regularization -> more flexible model.
- Smaller C -> stronger regularization -> simpler model.

---

## Suggested student comparison protocol

1. Fix one train/test split.
2. Change one setting at a time.
3. Record:
- setting change
- accuracy
- macro F1
- Precision@k
- notes from query walkthrough
4. Keep best setting only if validation/test improves.

---

## From by-hand learning to full pipeline

Start small:
- WORKED_NLP_MATH_EXAMPLE.md
- NLP_BY_HAND_WORKSHEET.md
- NLP_BY_HAND_WORKSHEET_ANSWER_KEY.md

Then scale up to:
- newsgroups_end_to_end_practice.py

This preserves intuition while moving to realistic data size and runtime workflows.
