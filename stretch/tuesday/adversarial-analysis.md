# Adversarial QA Probe — Analysis Memo

> Replace each placeholder section. Memo target: ~1 page. The TA rubric rewards specificity grounded in your data.

## 1. Hypothesis

State your targeted failure mode operationally:
- **Input pattern:**  The question asks for a short categorical label or noun
  (e.g., "What is X?" or "What is X's occupation?") while the context contains
  longer descriptive clauses that are semantically related but answer a
  different implicit question ("what does X do?" rather than "what IS X?")

- **Output pattern:**  The model returns a descriptive phrase or explanatory
  clause instead of the short semantic label targeted by the question. Example:
  gold answer `documentary`, predicted answer `America's health care system`

- **Why you hypothesize this:**  DistilBERT-SQuAD performs extractive span
  selection by aligning question and context representations. For semantic-type
  questions, the correct answer is often a short generic noun with weak lexical
  overlap with the question itself. Nearby explanatory clauses contain richer
  contextual information and stronger semantic associations, making them more
  attractive candidate spans. The span-selection objective does not explicitly
  prioritize short categorical labels over longer descriptive spans when both
  appear semantically plausible

## 2. Set Design

- Total examples: 33
- Tags used: 
    - `type-question` (15)
    - `genre-question` (8)
    - `role-question` (7)
    - `control` (3)
- Why these tags:
    - `type-question`: Core adversarial pattern testing whether the model selects
        descriptive clauses instead of short semantic labels.
    - `genre-question`: Tests whether highly standardized labels such as
        `comedy` or `science fiction` are easier for the model to extract.
    - `role-question`: Tests whether the model over-extends short occupational
        labels into compound descriptions.
    - `control`: Simple semantic-label questions without strong distractor
        clauses, used to isolate the effect of semantic competition
- Control examples: 3 controls (ADV_31, ADV_32, ADV_33). All three achieved
  EM = 1.0 and F1 = 1.0, confirming the model can correctly extract short
  semantic labels when contexts are unambiguous. This shows that the failure
  pattern is driven by nearby descriptive distractors rather than general task
  difficulty

## 3. Results

- Aggregate EM: 0.7576 ; Aggregate F1: 0.8405
- Lab 7B baseline (from your `qa_metrics.json`): EM 0.34; F1 0.46

The aggregate EM/F1 exceeded the Lab 7B baseline, which is expected — our
synthetic contexts were deliberately clean and extractive, making the task
easier overall. The diagnostic value lies in the per-pattern breakdown, which
partially confirms the hypothesis. The `type-question` pattern was the weakest
at EM = 0.60, while controls and genre labels both hit 1.0. However, the
failure rate on `type-question` was lower than expected, likely because the
synthetic contexts were not adversarial enough — the descriptive distractors
did not compete strongly enough with the short gold labels in every case.

- Per-pattern_tag breakdown:

| Pattern | n | EM | F1 | vs. baseline |
|---|---|---|---|---|
| type-question | 15 | 0.60 | 0.72 | +0.26 |
| genre-question | 8 | 1.00 | 1.00 | +0.66 |
| role-question | 7 | 0.71 | 0.86 | +0.37 |
| control | 3 | 1.00 | 1.00 | +0.66 |

Cite at least 3 specific (qid, question, gold, predicted) tuples that illustrate the patterns:

- **(ADV_01)** *What is Sicko?* → gold: `documentary`, predicted:
  `America's health care system`. The model extracted the topic discussed by
  the documentary rather than the semantic category requested by the question.

- **(ADV_03)** *What is the Kindle?* → gold: `e-reader device`, predicted:
  `to expand digital reading and wireless book delivery`. The model extracted
  the product's purpose rather than the categorical type label.

- **(ADV_25)** *What is Quincy Jones's occupation?* → gold: `producer`,
  predicted: `producer and composer`. The model identified the correct semantic
  region but over-extended the answer span into a compound occupational phrase.

## 4. Production Defense

The most targeted engineering defense is a **confidence-threshold filter**
that routes low-confidence predictions to a human reviewer instead of
automatically returning the model's answer.

This defense directly follows from the per-pattern results. The
`type-question` category produced the lowest EM/F1 in the set, indicating
that short categorical labels competing with descriptive clauses create
ambiguity during span selection. Even at a partial failure rate of 40%, an
incorrect categorical label returned with high confidence in a production
system — misidentifying a product type, genre, or occupation — could
meaningfully mislead users. A calibrated confidence threshold would surface
these uncertain predictions for human review without requiring retraining or
additional labeled data, making it the most practical immediate defense.
