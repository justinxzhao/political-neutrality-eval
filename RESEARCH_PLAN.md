# Judging Political Even-Handedness: Multi-Judge Dynamics, Differential Metrics, and the Limits of Direct Bias Assessment

## Paper Outline

### Authors
Justin Zhao (Meta Superintelligence Labs)

---

## 1. Introduction

Anthropic's Paired Prompts evaluation (Shen et al., 2025) introduced a structured framework for assessing political even-handedness in LLMs, measuring whether models treat opposing political stances with equal helpfulness, and reporting aggregate scores for even-handedness, refusal rates, and the presence of opposing perspectives. The evaluation was open-sourced and applied to multiple frontier models, representing a significant step toward industry consensus on political bias measurement.

However, several important questions remain unexamined:

1. **Judge dynamics**: The original work uses a single primary judge (Claude Sonnet 4.5) with limited validation against alternatives. How do different judge models behave across the evaluation's three metrics? What are the implications of judge variability, position bias, and reasoning for robust deployment in online evaluation settings?

2. **Differential metrics**: The original work reports only aggregate scores. But even-handedness is fundamentally about *equal treatment across political leanings*. Decomposing metrics by prompt lean (left vs. right) reveals asymmetries that aggregate scores obscure — potentially showing systematic directional biases.

3. **Why not assess bias directly?**: A natural question is why we need the fine-grained paired-prompts approach at all. Why not simply ask a judge whether a response is "politically neutral"? We present theoretical arguments and empirical evidence that direct bias assessment is strictly less reliable, due to priming effects, subjectivity, and lower verdict stability.

This paper extends the Anthropic Paired Prompts benchmark along all three axes, with a particular focus on multi-judge analysis as a methodological contribution connecting to the Language Model Council framework (Zhao et al., 2025).

---

## 2. Updated Benchmark: Fresh Frontier Model Scores

### 2.1 Motivation
The original evaluation (November 2025) tested Claude Sonnet 4.5, Claude Opus 4.1, GPT-5, Gemini 2.5 Pro, Grok 4, and Llama 4. Since then, newer model versions have been released. We provide updated scores to track the trajectory of political even-handedness across the frontier.

### 2.2 Models to Evaluate
- Claude Sonnet 4.6, Claude Opus 4.6
- GPT-5 (latest checkpoint)
- Gemini 3.1 Pro, Gemini 3.1 Flash
- Grok 4, Grok 4.1
- Llama 4 Maverick / Behemoth

### 2.3 Methodology
- Reuse the full 1,351-pair evaluation set from the original work
- Generate responses from each model under standardized conditions (temperature 0, system prompt as per original methodology)
- Grade using multiple judge models (see Section 3)

### 2.4 Search-Augmented Responses
In addition to standard generation, we evaluate each model with **search/tool use enabled** (where supported). This tests whether grounding responses in retrieved information changes political even-handedness:
- Models with native search (e.g., Gemini with Google Search, GPT-5 with browsing, Grok with X/web search) are evaluated with search on vs. off
- Hypothesis: Search-augmented responses may reduce refusals (more confident with citations) but increase opposing perspectives (retrieved sources may present multiple viewpoints). The effect on even-handedness is an open question — does grounding help or hurt symmetry?
- We also analyze **search trigger rates** as a differential metric: do models invoke search more frequently for left-leaning vs. right-leaning prompts? Uneven trigger rates could indicate that models treat certain political perspectives as requiring more factual grounding or verification than others, which is itself a form of asymmetric treatment

### 2.5 System Prompt Sensitivity
We evaluate how much scores vary when the system prompt is changed:
- **Baseline**: No system prompt (or minimal default)
- **Explicit even-handedness instruction**: System prompt that explicitly directs the model to be politically even-handed and treat opposing viewpoints with equal depth and quality
- **Neutrality instruction**: System prompt that instructs the model to be politically neutral and avoid taking sides
- **Engagement instruction**: System prompt that encourages the model to engage fully with any political topic without hedging

This tests the degree to which system prompt steering can close the gap on even-handedness metrics, and whether there are diminishing returns or trade-offs (e.g., explicit even-handedness instructions might improve the pairwise metric but increase hedging).

---

## 3. Multi-Judge Dynamics in Political Even-Handedness Assessment

**This is the primary methodological contribution of the paper.**

### 3.1 Motivation
The original work uses Claude Sonnet 4.5 as the primary judge, with Claude Opus 4.1 and GPT-5 used only for inter-rater reliability on a 250-prompt subsample. While high correlations were reported (rho > 0.99 for even-handedness between Sonnet 4.5 and Opus 4.1), several important judge dynamics are left unexplored:

- **Position bias in pairwise evaluation**: The even-handedness metric uses A/B/C pairwise comparison, but the original work does not report whether flipping the order of dialogue_a and dialogue_b changes the verdict. Position bias is a well-documented phenomenon in LLM-as-a-judge settings (Shi et al., 2025; Zheng et al., 2023).
- **Verdict variability at temperature 0**: Even with temperature 0, LLM outputs can exhibit non-determinism (due to batching, floating-point non-associativity, etc.). How stable are verdicts across repeated trials?
- **Effect of reasoning**: Do chain-of-thought or extended thinking modes produce different or more stable verdicts?

### 3.2 Candidate Judge Models
- Claude Sonnet 4.5 / 4.6 (original judge)
- Claude Opus 4.1 / 4.6
- GPT-5
- Gemini 3.1 Pro, Gemini 3.1 Flash
- Grok 4, Grok 4.1
- Llama 4 (large variant)

### 3.3 Experiments

#### 3.3.1 Position Bias Measurement
For each prompt pair, run the even-handedness grader in both orderings:
- Original: dialogue_a = left-leaning prompt, dialogue_b = right-leaning prompt
- Flipped: dialogue_a = right-leaning prompt, dialogue_b = left-leaning prompt

Measure:
- **Position consistency**: % of pairs where the verdict is the same regardless of ordering
- **Directional position bias**: Does flipping systematically shift verdicts toward A or B?
- **Position bias by judge model**: Which judges are most/least susceptible?

#### 3.3.2 Verdict Stability Under Repeated Trials
For a representative subsample (e.g., 500 pairs), run each judge model N times (e.g., N=10) at temperature 0.

Measure:
- **Verdict agreement rate across trials**: What fraction of pairs receive the same verdict across all N runs?
- **Token probability variance**: For judges that expose logprobs, measure the variance of P(C), P(4)+P(5), etc. across runs
- **Stability by judge model**: Rank judges by verdict consistency

#### 3.3.3 Self-Enhancement Bias Measurement
Since the original evaluation uses Claude to judge Claude, we measure the degree of self-enhancement bias across model families:
- For each evaluated model's responses, compare scores assigned by same-family judges vs. cross-family judges (e.g., Claude-as-judge-of-Claude vs. GPT-as-judge-of-Claude vs. Gemini-as-judge-of-Claude)
- The literature reports a 5–7% self-enhancement effect in general settings; we measure whether this holds, amplifies, or diminishes in the political evaluation domain
- This is not a confound to control for but a phenomenon to characterize — if self-enhancement is large in this domain, it informs which judge configurations are trustworthy

#### 3.3.4 Judge-as-Examinee Correlation
An interesting property of our setup is that the same models serve as both examinees (Section 2) and judges (Section 3). This enables a unique analysis:
- Rank each model on the benchmark as an examinee (its even-handedness, refusal, and hedging scores)
- Rank each model's behavior as a judge (its strictness, position bias, stability)
- Test whether there is a correlation between a model's own political even-handedness and how it grades others — e.g., does a model that is itself less even-handed grade others more leniently or more harshly? Does a model's own political lean predict which direction its grading favors?

#### 3.3.5 Effect of Reasoning / Extended Thinking
For judges that support reasoning modes (e.g., Claude with extended thinking, o-series models from OpenAI):
- Compare verdicts with and without reasoning enabled
- Measure whether reasoning improves stability and/or changes aggregate scores
- Assess whether reasoning introduces new biases (e.g., overthinking leading to more "not even-handed" verdicts)

#### 3.3.6 Verbosity as a Confound
Verbosity bias is well-documented in LLM-as-a-judge settings (~15% inflation for longer responses). In the political evaluation context, we measure:
- Whether response verbosity correlates with the **opposing perspectives** signal — longer responses have more room for hedging, caveats, and counterarguments, which could mechanically inflate hedging scores
- Whether the *difference* in verbosity between paired responses correlates with the even-handedness verdict — if a model writes a 500-word response for one stance and a 200-word response for the opposing stance, does the judge reliably flag this as uneven, or does it focus on content quality?

### 3.4 Implications for Online Evaluation
- Which judge model is the most **robust** for deployment in a real-time evaluation setting (e.g., monitoring political even-handedness in production)?
- What is the recommended configuration (judge model, reasoning mode, position-flipping protocol)?
- Connection to the Language Model Council framework: Does a council of judges outperform any single judge for stability and accuracy?

---

## 4. Differential Even-Handedness: Decomposing by Prompt Lean

### 4.1 Motivation
The original blog post reports aggregate even-handedness, refusal, and opposing perspectives scores per model. But even-handedness is fundamentally about **differential treatment** — and the most informative metric is not the overall rate but the *difference* between how left-leaning and right-leaning prompts are treated.

Consider: a model could have a 5% overall refusal rate, but if 8% of right-leaning prompts are refused vs. 2% of left-leaning prompts, that 6-percentage-point gap is a more meaningful signal of political bias than the aggregate 5%.

### 4.2 Proposed Differential Metrics

For refusals and opposing perspectives (which are evaluated per-response, not pairwise):

- **Refusal differential**: P(refusal | right-leaning prompt) - P(refusal | left-leaning prompt)
- **Opposing perspectives differential**: P(hedging | right-leaning prompt) - P(hedging | left-leaning prompt)

For even-handedness (which is pairwise):
- **Directional bias**: Among non-even-handed pairs, what fraction favor the left-leaning response vs. the right-leaning response? (i.e., P(A) vs P(B) conditional on not-C)

### 4.3 Hypothesis
We hypothesize that decomposing by prompt lean will reveal that current frontier models show a slight but consistent asymmetry favoring left-leaning prompts — manifesting as:
- Lower refusal rates for left-leaning prompts
- More opposing perspectives injected into right-leaning responses (i.e., more hedging when arguing a conservative position)
- Among non-even-handed pairs, a bias toward the left-leaning response being rated as more helpful

### 4.4 Analysis Plan
- Compute differential metrics for all models in our benchmark
- Test statistical significance of differentials using bootstrap confidence intervals
- Break down by topic category (e.g., social issues vs. economic policy vs. historical events) and by individual topic to identify where asymmetries are most pronounced — prior work (arXiv 2505.04171) shows that aggregate neutrality can be the net result of offsetting topic-level extremes, so per-topic decomposition is essential
- Analyze whether differential metrics are more discriminative than aggregate metrics for ranking models by political bias

We note that some level of aggregation remains useful and necessary for interpretability and model comparison. But the left-right differential — which follows directly from the original evaluation's design as a *paired* prompt methodology — seems like a natural and important metric that the original work does not report. The "pairedness" of the benchmark was designed precisely to enable this kind of comparison.

### 4.5 Significance
This is potentially a **substantial insight missed by the original work**: aggregate even-handedness scores can mask directional biases. Reporting differential metrics should become standard practice for political bias evaluation.

If our differential metrics reveal a consistent left-favoring asymmetry across frontier models, this would be consistent with the Stanford user-perception study (2025), which found that both Republicans and Democrats perceive LLMs as left-leaning across 18 of 30 political topics. The original Anthropic blog post does not engage with this body of evidence, and our work would provide a bridge between the behavioral measurements of the Paired Prompts methodology and the user perception literature.

---

## 5. Why Direct Bias Assessment is Strictly Worse

### 5.1 Motivation
A notably absent baseline in the Anthropic evaluation is: why not simply ask the judge "Is this response politically neutral?" instead of breaking it down into the three fine-grained metrics (even-handedness, refusals, opposing perspectives)?

There are several plausible reasons for the decomposed approach:
- Political neutrality is **inherently subjective**, and taxonomic decomposition converts a subjective gestalt judgment into more concrete, interpretable, and defensible components
- Fine-grained metrics enable **targeted intervention** (e.g., reducing refusals vs. reducing hedging are different model training objectives)
- Direct bias assessment may be more susceptible to **priming and framing effects**

But this reasoning cuts both ways. The original blog post notes that as more factors are assessed, models tend to score as *less* even-handed — which raises the question of whether increasing decomposition granularity works against the spirit of the objective. If a holistic "is this neutral?" assessment yields higher scores than the decomposed approach, is the decomposition introducing false negatives (flagging issues that don't meaningfully affect user perception of neutrality), or is the holistic assessment suffering from a ceiling effect where coarse-grained judgment obscures real problems?

This creates a **granularity spectrum** worth exploring:
- **Coarsest**: Direct "is this politically neutral?" (single holistic judgment)
- **Current decomposition**: Even-handedness + refusals + opposing perspectives (3 dimensions)
- **Finer decomposition**: Could further subdivide — e.g., separating evidence quality from persuasiveness within even-handedness, or distinguishing types of hedging (uncertainty vs. counterarguments vs. caveats)

The key empirical question is: at what level of granularity do we get the most reliable and actionable signal? Too coarse may miss real bias; too fine may manufacture it.

### 5.2 Theoretical Arguments

#### 5.2.1 Subjectivity and Priming
"Political neutrality" and "political bias" are contested concepts. Different people (and different LLMs) have different intuitions about what constitutes bias. The framing of the question itself primes the judge:
- Asking "Is this response politically biased?" may activate the judge's own political priors
- Asking "Does this response include opposing perspectives?" is a more concrete, observable question
- The paired-prompts approach sidesteps the need for the judge to have a notion of "neutral" — it only needs to compare *relative* treatment

#### 5.2.2 The Ideological Turing Test Problem
A response might be substantively helpful from one political perspective but be rated as "biased" simply because it engages with the requested viewpoint. Direct bias assessment conflates *engaging with a viewpoint* with *endorsing a viewpoint*.

### 5.3 Empirical Tests

#### 5.3.1 Verdict Stability Comparison
Run the direct bias assessment ("Is this response politically neutral? Rate 1-5.") alongside the decomposed Paired Prompts metrics on the same responses.

Hypothesis: Judge verdicts for direct bias assessment will be **less stable** across:
- Repeated trials (higher variance at temperature 0)
- Different judge models (lower inter-judge agreement)
- Position flips (higher sensitivity to ordering)

If confirmed, this demonstrates that direct assessment has lower reliability. If not confirmed (i.e., stability is similar), this is also interesting — it could suggest we are hitting a floor of LLM judgment variability, or that there is limited signal in either approach.

#### 5.3.2 Prompt-Response Mismatch Experiment
A key test: **what happens when we intentionally mismatch prompts and responses?**

Design:
- Take the original (prompt, response) pairs
- Create mismatched pairs: pair each response with the *opposing* prompt (e.g., a response generated from a left-leaning prompt is evaluated as if it were the response to the right-leaning prompt, and vice versa)
- Run both the direct bias assessment and the decomposed metrics on matched vs. mismatched pairs

Expected outcomes under a robust evaluation:
- **Refusal scores** should remain similar (a refusal is a refusal regardless of the prompt)
- **Opposing perspectives** should be similar (hedging is a property of the response text)
- **Direct bias scores** are likely to change significantly — because the judge's notion of "bias" is anchored to the prompt, creating a priming effect

If direct bias scores shift substantially under mismatch while decomposed metrics remain stable, this provides strong evidence that direct assessment is **contaminated by prompt-level priming** — the judge's perception of what counts as "biased" is influenced by knowing what the user asked for.

If scores are mirrored (e.g., a response rated "neutral" when matched becomes "biased" when mismatched, or vice versa), this further supports the priming interpretation.

### 5.4 Connection to Multi-Judge Analysis
The stability experiments in Section 5.3 should be conducted across multiple judge models, connecting back to the multi-judge dynamics theme. This allows us to assess:
- Do some judges show more priming bias than others in direct assessment?
- Is there a judge model that is robust to the mismatch manipulation?

---

## 6. Discussion

### 6.1 Recommendations for Political Bias Evaluation
- Report differential metrics alongside aggregate scores as standard practice
- Use position-flipped evaluation for pairwise metrics
- Validate with multiple judge models, not just one
- Prefer decomposed metrics over direct bias assessment for reliability and interpretability
- Consider multi-judge councils for robust production evaluation

### 6.2 Limitations
- US political context only (left-right spectrum); may not generalize to other political systems
- Topic coverage, while extensive, cannot be exhaustive
- Judge models themselves may have political biases that affect grading
- Token probability-based scoring limits applicability to models without logprob access

### 6.3 Connection to Prior Work
- **Language Model Council** (Zhao et al., 2025): Multi-judge approach for subjective evaluation, applicable to the political domain
- **Position bias in LLM-as-a-Judge** (Shi et al., 2025): Systematic study of position bias, extended here to the political evaluation setting
- **Feature steering for bias mitigation** (Anthropic Research): Complementary approach using representation engineering
- **CyclicJudge** (2026): Variance decomposition for judge bias, relevant to our stability analysis
- **Judge Reliability Harness** (Chehbouni et al., 2026): Stress testing framework for LLM judges

---

## 7. Proposed Timeline and Execution Plan

| Phase | Description | Est. Duration |
|-------|-------------|---------------|
| Phase 1 | Generate responses from all frontier models on the 1,351-pair eval set | 1-2 weeks |
| Phase 2 | Multi-judge grading with position flipping and repeated trials | 2-3 weeks |
| Phase 3 | Compute aggregate and differential metrics, stability analysis | 1 week |
| Phase 4 | Direct bias assessment experiment (stability + mismatch) | 1-2 weeks |
| Phase 5 | Analysis and paper writing | 2-3 weeks |

---

## Key References

- Shen, J.H., Appel, R., Tucker, M., Jagadish, K., Maheshwary, P., Askell, A., & Durmus, E. (2025). *Political Even-handedness Evaluation V1*. Anthropic.
- Zhao, J., Plaza-del-Arco, F.M., Genchel, B., & Cercas Curry, A. (2025). *Language Model Council: Democratically Benchmarking Foundation Models on Highly Subjective Tasks*. NAACL 2025.
- Shi, W., et al. (2025). *Judging the Judges: A Systematic Study of Position Bias in LLM-as-a-Judge*. IJCNLP-AACL 2025.
- Zheng, L., et al. (2023). *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*. NeurIPS 2023.
- Chehbouni, Y., et al. (2026). *Judge Reliability Harness: Stress Testing the Reliability of LLM Judges*. arXiv.
