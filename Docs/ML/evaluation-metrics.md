# ML evaluation metrics (quick reference)

Concise definitions, formulas, and tiny examples for common classification and NLP metrics.

---

## Accuracy

**Meaning:** Fraction of all predictions that are correct. Best when classes are balanced; misleading when one class dominates.

**Formula:**

$$
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
$$

**Example:** 90 correct out of 100 predictions → **Accuracy = 90/100 = 0.90 (90%)**.

---

## Precision

**Meaning:** Of everything the model predicted as positive, how many were actually positive. Answers: “When we say yes, how often are we right?” High precision → fewer false alarms.

**Formula:**

$$
\text{Precision} = \frac{TP}{TP + FP}
$$

**Example:** Model flags 10 emails as spam; 8 are truly spam, 2 are not.

- TP = 8, FP = 2 → **Precision = 8/(8+2) = 0.80**.

---

## Recall

**Meaning:** Of all actual positives, how many the model found. Answers: “Did we catch most of the real cases?” High recall → fewer missed positives (also called **sensitivity** or **true positive rate**).

**Formula:**

$$
\text{Recall} = \frac{TP}{TP + FN}
$$

**Example:** There are 12 real spam emails; the model catches 8 and misses 4.

- TP = 8, FN = 4 → **Recall = 8/(8+4) ≈ 0.67**.

**Precision vs recall:** Stricter threshold often raises precision and lowers recall; looser threshold does the opposite. **F1** balances both: $F1 = 2 \cdot \frac{P \cdot R}{P + R}$.

---

## BLEU (Bilingual Evaluation Understudy)

**Meaning:** Compares a **candidate** translation (or generation) to one or more **reference** texts using **n-gram overlap**, with a **brevity penalty** if the candidate is too short. Emphasizes **precision** of n-grams (modified to cap each n-gram’s contribution). Common in machine translation; BLEU ∈ [0, 1], higher is better.

**Formula (simplified):**

$$
\text{BLEU} = BP \cdot \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)
$$

- \(p_n\): modified n-gram precision (usually \(n = 1..4\), equal weights).
- \(BP\): brevity penalty if candidate length \(<\) reference length.

**Example:**

- Reference: `the cat is on the mat`
- Candidate: `the cat on the mat`

Unigrams and bigrams match well; missing **“is”** hurts longer n-grams. BLEU is **high but not 1.0** — good overlap, not identical.

---

## ROUGE (Recall-Oriented Understudy for Gisting Evaluation)

**Meaning:** Measures overlap between **generated** text and **reference** summary (or answer). **Recall-oriented** — “how much of the reference did we cover?” Variants: **ROUGE-N** (n-gram recall), **ROUGE-L** (longest common subsequence / sentence structure).

**Formula (ROUGE-N recall):**

$$
\text{ROUGE-N} = \frac{\sum_{S \in \{\text{refs}\}} \sum_{\text{gram}_n \in S} \text{Count}_{\text{match}}(\text{gram}_n)}{\sum_{S \in \{\text{refs}\}} \sum_{\text{gram}_n \in S} \text{Count}(\text{gram}_n)}
$$

(Implementation details vary by toolkit; idea is reference n-grams found in the hypothesis.)

**Example:**

- Reference: `the dog barked loudly`
- Summary: `the dog barked`

Reference unigrams `{the, dog, barked, loudly}` — hypothesis contains 3 of 4 → **unigram ROUGE recall ≈ 3/4 = 0.75**. Missing **“loudly”** lowers recall.

---

## Perplexity

**Meaning:** How “surprised” a **language model** is by a test sequence. Lower perplexity → the model assigns higher probability to the text (better fit on that data). Used for LM evaluation, not directly for “correct answers” in QA.

**Formula:**

$$
\text{Perplexity} = \exp\left(-\frac{1}{N}\sum_{i=1}^{N} \log P(w_i \mid w_{<i})\right) = 2^{H}
$$

where \(N\) is number of predicted tokens and \(H\) is average cross-entropy in bits (if log base 2).

**Example:** Model assigns probability 0.25 to each of 4 equally likely next words on average → $-\log_2(0.25) = 2$ bits/token → **perplexity = $2^2 = 4$**. If the model is sharper (higher probs on true tokens), perplexity **drops** (e.g. toward 1 for a perfect deterministic predictor on that text).

---

## When to use which

| Metric | Typical use |
|--------|-------------|
| Accuracy | Balanced classification |
| Precision / Recall | Imbalanced labels, search, fraud, medical screening |
| BLEU | Translation, strict n-gram similarity to reference |
| ROUGE | Summarization, overlap with reference text |
| Perplexity | Language model quality on a corpus |
