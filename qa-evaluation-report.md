# QA Evaluation Report — Lab 7B

## Dataset Description
~1,000 extractive QA examples drawn from a curated tech and entertainment
slice of the `glnmario/news-qa-summarization` dataset (CNN articles). Every
gold answer is a literal substring of its context by construction.

## Model
`distilbert-base-cased-distilled-squad`  
https://huggingface.co/distilbert-base-cased-distilled-squad

## Aggregate Metrics
| Metric | Score |
|--------|-------|
| Exact Match (EM) | 0.34 |
| Token F1 | 0.46 |

The 12-point gap between EM and F1 indicates the model frequently extracts
spans that overlap with the gold answer but differ at the boundaries — it
gets the right tokens but clips or extends the span, which EM penalizes
fully while F1 gives partial credit.

## Failure-Mode Taxonomy

### 1. Distractor Entity Selection
The model selects a named entity that appears in the context but belongs to
a different referent than what the question targets.

| Field | Value |
|-------|-------|
| qid | NEWS_0689_Q4 |
| Question | Who is the oldest supporting actor Oscar nominee ever? |
| Gold | Hal Holbrook |
| Predicted | Cassidy |

### 2. Wrong Semantic Type
The model returns a span of the wrong semantic category entirely — a clause
or phrase rather than the noun/label the question is asking for.

| Field | Value |
|-------|-------|
| qid | NEWS_0836_Q3 |
| Question | What is Sicko? |
| Gold | documentary |
| Predicted | who we put in power |

### 3. Distractor Number Selection
For numeric questions, the model picks a different number present in the
context rather than the one that answers the question.

| Field | Value |
|-------|-------|
| qid | NEWS_0408_Q2 |
| Question | How many Academy Awards was "Slumdog Millionaire" nominated for? |
| Gold | 10 |
| Predicted | three |

## Domain Judgment
I would not ship this model for **financial filings**. At 34% EM, the model
too frequently selects distractor numbers and entities from nearby context,
which in a financial setting could return the wrong revenue figure, share
count, or party name. The model also lacks no-answer support — it always
returns a span even when the context does not contain the answer — making it
unreliable for any workflow requiring detection of unanswerable queries.