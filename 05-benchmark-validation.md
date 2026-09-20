# Appendix D: Automated Benchmarking & LLM-as-a-Judge Validation

## D.1 Evaluator Configurations

Two LLM-as-a-judge configurations rated the quality of step-by-step
reasoning preceding an MMLU final answer.

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
given only in the required final-answer line is not penalized.

### D.1.1 Human rating instrument

Human reviewers applied the same three-level scale using the written
rubric reproduced in Appendix E. Its decision procedure caps at "To some
extent" any response whose reasoning is substantively correct but which
announces the selected option before the required `ANSWER: X` line. The
operational summary supplied to raters was: fully correct and relevant 
guidance without premature answer disclosure → Yes; partially useful guidance, 
limited errors, or premature disclosure → To some extent; no meaningful or 
predominantly incorrect guidance → No.

This is the pedagogy-focused criterion. Comparisons against the
content-focused automated evaluator therefore measure rubric divergence
in addition to rater disagreement, and are reported as such in §D.2.4.

### D.1.2 Response format and answer separability

Because the disclosure rule depends on whether the final answer is
separable from the guidance, each response was parsed to isolate the
benchmark-required `ANSWER: [LETTER]` line from the explanatory text.
Across the 1,199-response sample:

| Response structure | n | % |
|---|---|---|
| Answer given first; not separable | 547 | 45.6% |
| Final answer line cleanly separated | 329 | 27.4% |
| Answer appears inside explanation; not separable | 169 | 14.1% |
| Final line separated, plus additional disclosure | 95 | 7.9% |
| No separate final answer line detected | 59 | 4.9% |
| **Total** | **1,199** | **100%** |

Only 27.4% of responses place the answer solely in the required final
line. The remaining responses did not follow the cleanly separated 
final-answer format and were therefore evaluated for possible answer 
disclosure. When meaningful guidance also disclosed the selected answer,
the highest permissible rating was “To some extent.” This
structural fact accounts for the bulk of the disagreement documented in
§D.2.4 and §D.2.5.

## D.2 Human Validation of Automated Ratings

### D.2.1 Sample and exclusion flow

Human validation was conducted on an available subset of 960 responses 
drawn from a stratified random sample of 1,199 model outputs. The 
reviewed subset comprised 390 BLUECOMPUTER.2 responses, 388 
Qwen2.5-1.5B-Instruct responses, and 182 general.2 responses.

| Step | n |
|---|---|
| Responses in grading sample | 1,199 |
| Human guidance label supplied | 960 |
| Automated label returned `error` | −5 |
| **Analytic n** | **955** |

Human correctness marks were supplied for 948 responses; 947 had a valid
automated counterpart.

Coverage was not uniform across models:

| Model | Sampled | Human-rated | Coverage |
|---|---|---|---|
| BLUECOMPUTER.2 (constrained variant) | 400 | 390 | 97.5% |
| Qwen2.5-1.5B-Instruct (base) | 400 | 388 | 97.0% |
| general.2 (TOOL) | 399 | 182 | 45.6% |

For the initial automated evaluation, 955 responses had both a 
human rating and a valid automated rating; five automated outputs 
were unavailable. For the stricter Claude Sonnet 4.6 evaluation,
958 responses were comparable after excluding one safety refusal 
and one internally inconsistent judgment.

The human-validation sample was used to assess agreement between 
human and automated ratings. It was not intended to reproduce the 
exact guidance-label proportions observed across the complete 
14,042-item MMLU evaluation.

### D.2.2 Agreement

| Comparison | n | Exact | κ | 95% CI | Linear-wtd κ | Quadratic-wtd κ | Gwet's AC1 |
|---|---|---|---|---|---|---|---|
| **All models pooled** | 955 | 79.9% | 0.622 | [0.580, 0.667] | 0.646 | 0.684 | 0.727 |
| Qwen2.5-1.5B-Instruct | 386 | 94.8% | 0.897 | [0.854, 0.938] | 0.901 | 0.908 | 0.931 |
| general.2 (TOOL) | 179 | 68.2% | 0.417 | [0.305, 0.529] | 0.458 | 0.521 | 0.567 |
| BLUECOMPUTER.2 | 390 | 70.5% | 0.327 | [0.231, 0.415] | 0.369 | 0.440 | 0.624 |
| Conditional: answer incorrect | 503 | 81.5% | 0.624 | [0.556, 0.690] | 0.633 | 0.650 | 0.756 |
| Conditional: answer correct | 452 | 78.1% | 0.616 | [0.552, 0.678] | 0.654 | 0.710 | 0.694 |

Because the three guidance categories are ordinal, we additionally 
report linear-weighted Cohen’s \(\kappa\), which assigns less weight 
to disagreements between adjacent categories than to disagreements 
between “Yes” and “No.”

Confidence intervals are percentile bootstrap, 2,000 resamples.
Disagreements were almost entirely one step on the ordinal scale (20.0%
of cases) rather than two (0.1%), so raters rarely inverted a judgment
outright.

Pooled agreement is materially higher than agreement on either
fine-tuned model. Averaging the three per-model κ values with equal
weight gives 0.547, against the pooled 0.622: the pooled figure is
lifted by the base model, whose outputs are predominantly answer-only
and therefore trivially classified. Per-model figures should be read in
preference to the pooled row.

### D.2.3 Confusion matrix

Rows are human labels, columns automated (n = 955):

| | No | To some extent | Yes | Total |
|---|---|---|---|---|
| **No** | 316 | 20 | 0 | 336 |
| **To some extent** | 86 | 443 | 45 | 574 |
| **Yes** | 1 | 40 | 4 | 45 |
| **Total** | 403 | 503 | 49 | 955 |

### D.2.4 Category-level agreement

| Label | n (human) | n (automated) | Agreed | Recall | Precision |
|---|---|---|---|---|---|
| No | 336 | 403 | 316 | 0.940 | 0.784 |
| To some extent | 574 | 503 | 443 | 0.772 | 0.881 |
| **Yes** | **45** | **49** | **4** | **0.089** | **0.082** |

Agreement on "No" and "To some extent" is strong. Agreement on "Yes" 
was very low: of 45 responses humans rated Yes, the automated evaluator
concurred on 4; of 49 it rated Yes, humans concurred on 4. Pooled κ
conceals this because "Yes" constitutes only 4.7% of human labels.

Collapsing the scale to a binary — any meaningful guidance versus
answer-only — yields substantially better agreement, because that
contrast does not depend on the disclosure rule:

| Model | n | Exact | κ |
|---|---|---|---|
| All pooled | 955 | 88.8% | 0.765 |
| Qwen2.5-1.5B-Instruct | 386 | 97.9% | 0.958 |
| general.2 (TOOL) | 179 | 78.2% | 0.556 |
| BLUECOMPUTER.2 | 390 | 84.6% | 0.522 |

### D.2.5 Directional bias

The automated evaluator assigned lower ratings than the human reviewers 
in 127 of the 192 disagreements and higher ratings in 65. This 
asymmetry was concentrated in general.2, for which the human rating was 
higher in 56 of 57 disagreements. Ratings for BLUECOMPUTER.2 were 
approximately balanced in direction.

Because the stricter human criteria generally assigned *higher* ratings
than the content-focused automated evaluator, These results indicate 
systematic evaluator differences, particularly for general.2; consequently, 
human and automated ratings are reported separately.

### D.2.6 Correctness parsing

Human correctness marks agreed with the automated `ANSWER: X` parse on
920 of 947 responses (97.1%; 27 disagreements), supporting the
extraction used throughout the benchmark.

## D.3 MMLU Knowledge-Preservation Check

Full benchmark, n = 14,042 per model:

| Model | Humanities | STEM | Social Sci. | Other | **Overall** |
|---|---|---|---|---|---|
| Qwen2.5-1.5B-Instruct (base) | 0.4839 | 0.5742 | 0.6331 | 0.5873 | **0.5599** |
| general.2 (TOOL) | 0.4993 | 0.4911 | 0.6555 | 0.6116 | **0.5577** |
| BLUECOMPUTER.2 | 0.4633 | 0.4887 | 0.5707 | 0.5688 | **0.5167** |

TOOL aggregate accuracy was 0.22 percentage points lower than that of the 
base model. The similar aggregate scores suggest that fine-tuning did not 
materially reduce performance on the evaluated MMLU questions.


## D.4 Model Identification and Guidance Ratio

Table 1 uses abbreviated model identifiers. Model1 denotes BLUECOMPUTER.2, 
the more constrained model variant included only in the automated benchmark. 
Model2 denotes general.2 (\tool{}), the model deployed in the educator 
evaluation. BLUECOMPUTER.2 was not used in the educator evaluation.

- **Model1 = BLUECOMPUTER.2** — constrained variant; not used in the expert evaluation (§3.2)
- **Model2 = general.2 = TOOL** — the system evaluated with educators

Guidance improvement over the base model, on the 1,199-response sample:

| Model | "Yes" (auto) | Ratio | "Yes" (human) | Ratio | Any guidance (human) | Ratio |
|---|---|---|---|---|---|---|
| Qwen2.5-1.5B-Instruct | 0.018 | 1.00 | 0.018 | 1.00 | 0.433 | 1.00 |
| general.2 (TOOL) | 0.192 | 10.9 | 0.104 | 5.8 | 0.714 | 1.65 |
| BLUECOMPUTER.2 | 0.108 | 6.1 | 0.049 | 2.7 | 0.828 | 1.91 |

TOOL versus base, human labels, Fisher's exact test:

- Any meaningful guidance: 130/182 (71.4%) vs 168/388 (43.3%); ratio 1.65, p = 3.8 × 10⁻¹⁰
- "Yes" rating: 19/182 (10.4%) vs 7/388 (1.8%); ratio 5.79, p = 1.3 × 10⁻⁵

Both contrasts are significant and both favor the fine-tuned model. They
differ in how well the validation supports them: the binary contrast
rests on a scale collapse with κ = 0.765 pooled, whereas the "Yes"
contrast rests on the one category where human and automated raters
agree at chance (§D.2.4).

The 12.7-fold improvement reported in the main text was calculated from the 
proportion of responses receiving a “Yes” rating in the complete automated 
benchmark. Ratios calculated from the smaller human-validation sample differ 
because of sampling variation, uneven human-review coverage, and differences 
between human and automated rating behavior. The validation subset was used 
to assess human–LLM agreement rather than to reproduce the full-corpus effect 
estimate.

## D.5 Limitations of This Validation

1. **Uneven coverage.** Human labels cover 97.5% of BLUECOMPUTER.2 and
   97.0% of the base model but only 45.6% of TOOL. Pooled statistics are
   therefore weighted away from the system the paper evaluates.
2. **Rubric divergence.** The human instrument penalizes answer
   disclosure; the content-focused automated rubric does not. Since only
   27.4% of responses place the answer solely in the final line
   (§D.1.2), the reported agreement conflates rater disagreement with
   criterion divergence. It is a lower bound on achievable agreement
   between raters applying the same rubric.
3. **"Yes" agreement is at chance**, and the benchmark headline is a
   ratio of "Yes" rates.
