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

Human validation was conducted on an available subset of 1,140 responses 
drawn from a stratified random sample of 1,199 model outputs. The 
reviewed subset comprised 390 BLUECOMPUTER.2 responses, 388 
Qwen2.5-1.5B-Instruct responses, and 182 general.2 responses.

| Step | n |
|---|---|
| Responses in grading sample | 1,199 |
| Human guidance label supplied | 1,140 |
| Automated label returned `error` | −5 API overload errors  |
| **Analytic n** | **1,135** |

Human correctness marks were supplied for 948 responses; 947 had a valid
automated counterpart.

Coverage was not uniform across models:

| Model | Sampled | Human-rated | Coverage |
|---|---|---|---|
| BLUECOMPUTER.2 (constrained variant) | 400 | 390 | 97.5% |
| Qwen2.5-1.5B-Instruct (base) | 400 | 388 | 97.0% |
| general.2 (TOOL) | 399 | 362 | 90.7% |

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
| **All models pooled** | 1,135 | 75.1% | 0.544 | 0.585 | 0.647 | 0.657 |
| Qwen2.5-1.5B-Instruct | 386 | 94.8% | 0.897 | [0.853, 0.937] | 0.901 | 0.908 | 0.931 |
| general.2 (TOOL) | 359 | 58.8% | 0.249 | [0.166, 0.366] | 0.330 | 0.447 | 0.434 |
| BLUECOMPUTER.2 | 390 | 70.5% | 0.327 | [0.230, 0.420] | 0.369 | 0.440 | 0.624 |
| Conditional: answer incorrect | 554 | 81.4% | 0.607 | [0.540, 0.675] | 0.617 | 0.637 | 0.757 |
| Conditional: answer correct | 564 | 68.3% | 0.475 | [0.411, 0.537] | 0.548 | 0.644 | 0.547 |

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
weight gives 0.544, against the pooled 0.584: the pooled figure is
lifted by the base model, whose outputs are predominantly answer-only
and therefore trivially classified. Per-model figures should be read in
preference to the pooled row.

### D.2.3 Confusion matrix

Rows are human labels, columns automated (n = 1,135):

| | No | To some extent | Yes | Total |
|---|---|---|---|---|
| **No** | 314 | 44 | 1 | 359 |
| **To some extent** | 89 | 517 | 98 | 704 |
| **Yes** | 0 | 51 | 21 | 72 |
| **Total** | 403 | 612 | 120 | 1,135 |

### D.2.4 Category-level agreement

| Label | n (human) | n (automated) | Agreed | Recall | Precision |
|---|---|---|---|---|---|
| No | 359 | 403 | 314 | 0.875 | 0.779 |
| To some extent | 704 | 612 | 517 | 0.734 | 0.845 |
| **Yes** | **72** | **120** | **21** | **0.292** | **0.175** |

Agreement on “Yes” remained substantially weaker than agreement on the 
other categories. The automated evaluator identified only 21 of the 72 
human “Yes” ratings and humans confirmed only 21 of the evaluator’s 120 
“Yes” ratings. Collapsing the scale to a binary — any meaningful 
guidance versus answer-only — yields substantially better agreement, 
because that contrast does not depend on the disclosure rule:

| Model | n | Exact | κ |
|---|---|---|---|
| All pooled | 1,135 | 88.2% | 0.736 |
| Qwen2.5-1.5B-Instruct | 386 | 97.9% | 0.957 |
| general.2 (TOOL) | 359 | 81.6% | 0.466 |
| BLUECOMPUTER.2 | 390 | 84.6% | 0.522 |

### D.2.5 Directional bias

For the original content-focused evaluation, disagreement direction was
nearly balanced: Claude assigned a lower category than the human reviewer
in 140 cases and a higher category in 143 cases.

The revised pedagogy-focused evaluator was systematically more
conservative. It assigned a lower category than the human reviewer in
395 cases and a higher category in 49 cases. Ten disagreements crossed
directly between “No” and “Yes”; the remaining disagreements involved
adjacent categories.

### D.2.6 Correctness parsing

Usable human correctness marks were available for 1,126 responses.
Human correctness judgments agreed with the automated answer parse on
1,026 responses (91.1%), with 100 disagreements. Seventy-two correctness
cells were blank, and one ambiguous entry (`TRUE/FLASE`) was excluded.

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
| Qwen2.5-1.5B-Instruct | 0.0175 | 1.00 | 0.018 | 1.00 | 0.433 | 1.00 |
| general.2 (TOOL) | 0.1905 | 10.88 | 0.1271 | 7.04 | 0.793 | 1.83 |
| BLUECOMPUTER.2 | 0.1075 | 6.14 | 0.0487 | 2.7 | 0.828 | 1.91 |

TOOL versus base, human labels, Fisher's exact test:

- Any meaningful guidance: 286/362 = (79.3%) vs 168/388 (43.3%); ratio 1.83, p = 1.66 × 10⁻²⁴
- "Yes" rating: 46/362 = (12.7%) vs 7/388 (1.8%); ratio 7.04, p = 1.99 × 10⁻⁹

Both contrasts are significant and both favor the fine-tuned model. They
differ in how well the validation supports them: the binary contrast
rests on a scale collapse with κ = 0.765 pooled, whereas the "Yes"
contrast rests on the one category where human and automated raters
agree at chance (§D.2.4).

| Model | Total attempted | Evaluator errors | Valid evaluations | Automated “Yes,” n | Automated “Yes” rate | Ratio vs. Qwen |
|---|---:|---:|---:|---:|---:|---:|
| Qwen2.5-1.5B-Instruct | 14,042 | 66 | 13,976 | 217 | 0.0155 (1.55%) | 1.00 |
| general.2 (\tool{}) | 14,042 | 120 | 13,922 | 2,677 | 0.1923 (19.23%) | 12.38 |
| BLUECOMPUTER.2 | 14,042 | 0 | 14,042 | 1,506 | 0.1072 (10.72%) | 6.91 |

The 12.4-fold improvement reported in the main text was calculated from the 
proportion of responses receiving a “Yes” rating in the complete automated 
benchmark. Ratios calculated from the smaller human-validation sample differ 
because of sampling variation, uneven human-review coverage, and differences 
between human and automated rating behavior. The validation subset was used 
to assess human–LLM agreement rather than to reproduce the full-corpus effect 
estimate.

## D.5 Limitations of This Validation

1. **Incomplete human coverage.** Human guidance labels were available
   for 95.1% of the sampled responses. Missing labels were concentrated
   in general.2 (37 missing), compared with 10 for BLUECOMPUTER.2 and
   12 for Qwen.
2. **Rubric divergence.** Human reviewers applied the pedagogy-focused
   disclosure rule, whereas the original automated evaluation emphasized
   conceptual content. Their agreement therefore measures both rating
   consistency and differences between the rubrics. The revised Claude
   evaluation more closely matched the human rubric but applied it more
   conservatively.
3. **Rare top category.** Only 72 comparable responses received a human
   “Yes” rating. Class-specific conclusions about “Yes” are therefore
   less stable than conclusions about the broader distinction between
   meaningful guidance and no guidance.
