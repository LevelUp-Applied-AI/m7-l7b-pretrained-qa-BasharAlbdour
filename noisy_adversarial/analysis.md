# Noisy Adversarial QA Probe — Robustness Analysis

## 1. Objective

This exploratory extension evaluates whether the semantic-type extraction
failures identified in the original adversarial QA probe become more severe
under noisy human-style text containing grammatical inconsistencies,
informal phrasing, and spelling mistakes.

The experiment keeps the same core hypothesis:
the QA model struggles to extract short categorical labels when nearby
descriptive clauses are semantically richer and more informative.

However, unlike the original clean adversarial set, this version introduces:

- grammar mistakes,
- spelling mistakes,
- tense inconsistencies,
- missing punctuation,
- and more natural human-written phrasing.

The dataset was intentionally designed to resemble informal Wikipedia-style
summaries, football discussions, engineering forums, and entertainment
descriptions written quickly by a human rather than benchmark-clean English.

---

## 2. Hypothesis

- **Input pattern:** Contexts contain noisy human-authored language alongside
  semantic-category questions such as:
  - “what kinda league is la liga suppose to be?”
  - “spotify can be categorized as what type of service?”
  - “discord nowadays is considered what sort of platform?”

  The gold answer is usually a short categorical label while nearby text
  contains longer descriptive semantic content.

- **Output pattern:** The model extracts:
  - incomplete semantic labels,
  - descriptive explanatory clauses,
  - truncated categories,
  - or over-extended semantic spans
  instead of the concise categorical answer.

- **Why this likely occurs:** DistilBERT-SQuAD performs extractive span
  selection rather than semantic normalization. Under noisy text conditions,
  token boundaries and grammatical structure become less reliable, making it
  harder for the model to isolate compact noun phrases. The model still
  identifies the correct semantic region but frequently fails to determine the
  precise categorical boundary.

---

## 3. Results

### Aggregate Metrics

| Metric | Score |
|---|---|
| EM | 0.67 |
| F1 | 0.87 |
| n | 33 |

### Per-Pattern Results

| Pattern | n | EM | F1 |
|---|---|---|---|
| type-question | 15 | 0.60 | 0.86 |
| genre-question | 6 | 1.00 | 1.00 |
| role-question | 6 | 0.50 | 0.72 |
| control | 6 | 0.67 | 0.93 |

---

## 4. Key Findings

### 4.1 The model remained semantically robust to moderate spelling noise

Despite introducing grammar mistakes and spelling errors such as:

- `proffesional`
- `platfrom`
- `renovtions`
- `extreemly`
- `relased`
- `populer`

the model usually preserved semantic understanding of the context.

The results show that moderate human-style spelling noise did not cause
catastrophic comprehension failure.

Instead, the model typically remained semantically close to the gold answer.

This is supported by the relatively high overall F1 score (0.87).

---

### 4.2 The primary weakness remained semantic-boundary extraction

Most failures were not random predictions.
Instead, they reflected semantic truncation or boundary drift.

Representative examples:

| Gold | Predicted |
|---|---|
| `voice and text communication platform` | `voice and text communication` |
| `music-streaming service` | `music-streaming` |
| `professional football club` | `professional football` |
| `DevOps platform` | `DevOps` |

These predictions show that the model often located the correct semantic
region but failed to extract the complete categorical label.

---

### 4.3 The model preferred descriptive semantic content over concise labels

The strongest adversarial failure remained:

- **Question:** `arduino is described as what exactly?`
- **Gold:** `open-source electronics platform`
- **Predicted:** `a project for students and hobbyists who needed affordable electronics tools`

Here the model ignored the concise semantic category and instead extracted a
long explanatory clause from nearby context.

This strongly supports the original semantic-type confusion hypothesis.

---

### 4.4 Spelling mistakes did not significantly worsen semantic extraction

Interestingly, the aggregate EM score remained unchanged from the previous
noisy experiment, while F1 increased slightly.

This suggests that spelling mistakes mainly affected:
- exact boundary precision,
- lexical matching,
- and span completeness,

rather than high-level semantic understanding.

The model remained capable of identifying the correct semantic neighborhood
even under noisy human-style text.

---

## 5. Interpretation

The experiment suggests that DistilBERT-SQuAD is moderately robust to natural
human spelling and grammar noise.
However, noisy language amplifies existing weaknesses in semantic-category
extraction and span-boundary precision.

The dominant failure mode is therefore not:
- semantic misunderstanding,

but rather:
- semantic granularity confusion.

The model frequently understands *what concept* the answer refers to while
failing to extract the exact categorical span requested by the question.

---

## 6. Production Implications

These findings are important for real-world QA deployments because production
inputs rarely resemble benchmark-clean English.

User-generated text often contains:
- spelling mistakes,
- informal phrasing,
- grammatical inconsistencies,
- and fragmented sentence structures.

Although the model remained relatively robust semantically, the experiment
shows that noisy text increases the likelihood of:
- truncated labels,
- incomplete entity types,
- and over-extended descriptive spans.

Applications requiring precise categorical answers (e.g., medical labels,
financial entity types, legal roles, or technical classifications) would
therefore benefit from:
- confidence-threshold filtering,
- answer-length normalization,
- or post-processing constraints on expected semantic answer types.