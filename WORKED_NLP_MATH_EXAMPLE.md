# Worked NLP Math Example (By Hand)

This mini example is intentionally small so students can do the math by hand.

## Tiny corpus (3 sentences)

D1: I love NLP, NLP is fun.
D2: I love pizza and coding.
D3: Coding in Python is fun.

Assume cleaning rules:
- lowercase
- remove punctuation
- remove stopwords: i, is, and, in

## Step 1) Cleaning output

D1 -> love nlp nlp fun
D2 -> love pizza coding
D3 -> coding python fun

## Step 2) Vocabulary

Sort unique tokens alphabetically:

[coding, fun, love, nlp, pizza, python]

Index map:
- coding -> 0
- fun -> 1
- love -> 2
- nlp -> 3
- pizza -> 4
- python -> 5

## Step 3) Bag-of-Words matrix (counts)

Rows are documents, columns follow vocabulary order.

D1: [0, 1, 1, 2, 0, 0]
D2: [1, 0, 1, 0, 1, 0]
D3: [1, 1, 0, 0, 0, 1]

Matrix X:

[0, 1, 1, 2, 0, 0]
[1, 0, 1, 0, 1, 0]
[1, 1, 0, 0, 0, 1]

## Step 4) TF-IDF by hand

Let N = 3 documents.
Use smoothed idf:

idf(t) = ln((N + 1) / (df(t) + 1)) + 1

Document frequency df:
- coding: 2 (D2, D3)
- fun: 2 (D1, D3)
- love: 2 (D1, D2)
- nlp: 1 (D1)
- pizza: 1 (D2)
- python: 1 (D3)

So:
- idf for df = 2: ln(4/3) + 1 = 1.2877
- idf for df = 1: ln(4/2) + 1 = 1.6931

Example TF choices (normalized by document length):
- D1 length = 4 tokens
- tf(nlp, D1) = 2/4 = 0.5

Then:
- tfidf(nlp, D1) = 0.5 * 1.6931 = 0.8466
- tf(fun, D1) = 1/4 = 0.25
- tfidf(fun, D1) = 0.25 * 1.2877 = 0.3219

Students can compute the remaining entries similarly.

## Step 5) Retrieval with cosine similarity

Query: love fun
Query vector q (BoW order): [0, 1, 1, 0, 0, 0]

Cosine(q, D1):
- q dot D1 = 2
- ||q|| = sqrt(2)
- ||D1|| = sqrt(6)
- cosine = 2 / (sqrt(2) * sqrt(6)) = 0.577

Cosine(q, D2):
- q dot D2 = 1
- ||D2|| = sqrt(3)
- cosine = 1 / (sqrt(2) * sqrt(3)) = 0.408

Cosine(q, D3):
- q dot D3 = 1
- ||D3|| = sqrt(3)
- cosine = 0.408

Ranking: D1 first, then D2 and D3 tied.

## Step 6) One manual classification score

Suppose we want class "tech" with a toy linear model:

z = w^T x + b

Use weights in vocabulary order:
- w = [0.2, 1.0, 0.1, 0.8, -0.9, 0.7]
- b = -0.2

For D3, x = [1, 1, 0, 0, 0, 1]

z = 0.2(1) + 1.0(1) + 0.1(0) + 0.8(0) - 0.9(0) + 0.7(1) - 0.2
z = 1.7

If using sigmoid for binary probability:

p = 1 / (1 + exp(-z)) = 0.8455

Interpretation: high confidence for class "tech" in this toy setup.

## Step 7) One KMeans iteration (k = 2)

Initialize centroids with D1 and D2.
Assign D3 to nearest centroid using squared Euclidean distance.

Distance from D3 to D1:
- ||D3 - D1||^2 = 7

Distance from D3 to D2:
- ||D3 - D2||^2 = 4

So D3 joins cluster of D2.

Updated centroid for cluster containing D2 and D3:
- mu2 = (D2 + D3) / 2 = [1, 0.5, 0.5, 0, 0.5, 0.5]

That is one full assign-update cycle.

## How this maps to project code

- Cleaning module: newsgroups_cleaning.py
- Classification route: newsgroups_classification_route.py
- Clustering route: newsgroups_clustering_route.py
- Retrieval route: newsgroups_retrieval_route.py
- Main orchestrator: newsgroups_end_to_end_practice.py

Students should do this tiny example first, then run the full pipeline to scale the same concepts.
