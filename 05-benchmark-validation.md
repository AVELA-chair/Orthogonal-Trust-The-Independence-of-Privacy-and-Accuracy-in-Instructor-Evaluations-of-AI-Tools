# Appendix D: Automated Benchmarking & LLM-as-a-Judge Validation

## D.1 Evaluator Configurations

Two LLM-as-a-judge configurations were used to rate the quality of
step-by-step reasoning preceding an MMLU final answer, with different
rubric stringency:

**Content-focused rubric (Llama 3.1 70B Instruct)**
- Score 3 ("Yes"): accurate explanations, sound reasoning, teaches correct concepts
- Score 2 ("To some extent"): has explanations but contains errors or misconceptions
- Score 1 ("No"): just states the answer without explanation

**Pedagogy-focused rubric (Claude Sonnet 4.6, stricter)**
- Score 3 ("Yes"): accurate explanations, sound reasoning, correct concepts, does *not* provide the complete answer
- Score 2 ("To some extent"): has explanations but contains errors, misconceptions, or gives away the complete answer
- Score 1 ("No"): no meaningful guidance (e.g., only a letter answer)

The pedagogy-focused rubric treats disclosing the answer as a limitation
even when the surrounding explanation is otherwise correct; an answer
given only in the required final-answer line (as instructed by the MMLU
prompt format) is not penalized.

## D.2 Human Validation of Automated Ratings

Human reviewers applied the same three-level scale to an available subset
of 960 responses:

| Evaluator | Exact agreement with human raters | n | Cohen's $\kappa$ (unweighted) | Linear-weighted Cohen's $\kappa$ |
|---|---|---|---|---|
| Content-focused (Llama 3.1 70B) | 79.9% | 955 | 0.622 | 0.646 |
| Pedagogy-focused (Claude Sonnet 4.6) | 59.9% | 958 | 0.317 | 0.344 |

(n differs slightly per evaluator after excluding one safety refusal and
one internally inconsistent judgment from the pedagogy-focused run.)

Because the stricter pedagogy-focused evaluator generally assigned lower
ratings than human reviewers, we treat its results as a conservative
sensitivity analysis relative to the content-focused evaluation, which
shows substantially higher agreement with human judgment.

## D.3 MMLU Knowledge-Preservation Check

The fine-tuned model and its unmodified base model were compared on MMLU
(Massive Multitask Language Understanding) across subjects spanning STEM
and humanities at difficulty levels from high school to college. The
fine-tuned model scored 55.7% versus 55.9% for the base model — within
0.2 points — indicating that the instructional fine-tuning did not
materially reduce general domain knowledge.
