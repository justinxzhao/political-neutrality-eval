# Prior Work: Political Neutrality Evaluation and Related Research

A survey of work related to Anthropic's Political Even-handedness Evaluation (Shen et al., 2025) and the research directions outlined in [RESEARCH_PLAN.md](RESEARCH_PLAN.md).

---

## 1. The Original Anthropic Work

### Measuring Political Bias in Claude (Shen et al., Nov 2025)
- **Blog post**: [Anthropic — Measuring political bias in Claude](https://www.anthropic.com/news/political-even-handedness)
- **GitHub**: [anthropics/political-neutrality-eval](https://github.com/anthropics/political-neutrality-eval)
- **Appendix**: [PDF — Appendix to Measuring political bias in Claude](https://assets.anthropic.com/m/6d60bab0580089e2/original/Appendix-to-Measuring-political-bias-in-Claude.pdf)

Introduced the **Paired Prompts** method: 1,351 prompt pairs across 150 topics and 7 topic types, graded on three dimensions (even-handedness, refusals, opposing perspectives) using Claude Sonnet 4.5 as the primary judge with token probability extraction. Key results: Claude Opus 4.1 (95%), Claude Sonnet 4.5 (94%), Gemini 2.5 Pro (97%), Grok 4 (96%), GPT-5 (89%), Llama 4 (66%).

**Appendix findings on grader discrepancies**: GPT-5 sometimes rated pairs as even-handed when one response appeared helpful and neutral but failed to engage with the actual request, or when both sides received helpful answers but with notable imbalances in persuasion and clarity. Claude placed greater emphasis on comparable substance and presentation quality across the pair. The opposing perspectives threshold was adjusted from 0.5 to 0.1 when calibrating against GPT-5 judgments.

**Relevance to our research plan**: This is the primary work we are extending. Key gaps we identified: no position flipping for the pairwise metric, no differential metrics by prompt lean, no exploration of judge dynamics beyond inter-rater reliability, no direct bias assessment baseline, no search-augmented or system-prompt-varied evaluations.

**Notable**: Ruth Appel, a co-author on this work, is also a co-author on the ICML 2025 position paper on political neutrality (see Section 2 below), suggesting ongoing research interest within Anthropic on these questions.

---

## 2. Political Neutrality: Theoretical Foundations

### Political Neutrality in AI Is Impossible — But Here Is How to Approximate It (Fisher, Appel, et al., ICML 2025 — Oral)
- **Paper**: [arXiv:2503.05728](https://arxiv.org/abs/2503.05728)
- **OpenReview**: [openreview.net/forum?id=H72JEXAPwo](https://openreview.net/forum?id=H72JEXAPwo)
- **Stanford HAI brief**: [hai.stanford.edu/policy/toward-political-neutrality-in-ai](https://hai.stanford.edu/policy/toward-political-neutrality-in-ai)
- **Code**: [github.com/jfisher52/Approximation_Political_Neutrality](https://github.com/jfisher52/Approximation_Political_Neutrality)

A position paper arguing that true political neutrality is neither feasible nor universally desirable, but that practical approximation remains essential. Proposes eight techniques across three levels: **output** (e.g., refusal, balanced presentation), **system** (e.g., fine-tuning, system prompts), and **ecosystem** (e.g., diversity of models). Inspired by Raz's philosophical insight that "neutrality can be a matter of degree."

**Relevance to our research plan**: Directly motivates Section 5 (why direct bias assessment is brittle) and Section 2.5 (system prompt sensitivity). If neutrality is fundamentally subjective and a matter of degree, then asking a judge "is this neutral?" inherits all that subjectivity. The decomposed approach in Paired Prompts can be seen as one of the "approximation techniques" this paper advocates — breaking a subjective gestalt into more concrete, measurable proxies. Also relevant to the granularity spectrum question: at what decomposition level do we get the best signal?

---

## 3. Paired/Contrastive Prompt Methodologies

### TwinViews: Topically Matched Paired Statements (Rains et al., EMNLP 2024)
- **Paper**: [On the Relationship between Truth and Political Bias](https://aclanthology.org/2024.emnlp-main.508.pdf)

Created TwinViews-13k, a dataset of 13,855 pairs of left-leaning and right-leaning statements matched by topic. Used GPT-3.5 Turbo to generate pairs kept "as similar as possible in style and length." Key finding: "truthful" reward models usually show a left-leaning political bias.

**Relevance**: Similar paired-prompt design to Anthropic's approach but focused on truthfulness rather than even-handedness. The finding that truth-evaluating models show left-leaning bias is relevant to our differential metrics analysis.

### PolBiX: Minimal Pairs for Fact-Checking (2025)
- **Paper**: [PolBiX: Detecting LLMs' Political Bias in Fact-Checking through X-phemisms](https://arxiv.org/pdf/2509.15335)

Constructs minimal pairs of factually equivalent claims differing in political connotation to assess consistency of LLMs in classifying them as true or false. Found that "the presence of judgmental words significantly influences truthfulness assessment" and that bias "is not mitigated by explicitly calling for objectivism in prompts."

**Relevance**: Directly relevant to our system prompt sensitivity experiments (Section 2.5) — explicit objectivity instructions did not help here, which may also be the case for even-handedness instructions.

### OpenAI's Multi-Axis Bias Framework (2025)
- **Blog**: [Defining and evaluating political bias in LLMs](https://openai.com/index/defining-and-evaluating-political-bias-in-llms/)

Evaluates ~500 prompts spanning 100 topics across neutral, slightly slanted, and emotionally charged prompts. Measures five axes of bias including asymmetric coverage and political refusals. Found models stay near-objective on neutral prompts but exhibit moderate bias on emotionally charged ones. GPT-5 reduced bias scores by ~30% compared to prior models.

**Relevance**: Complementary evaluation framework. Their multi-axis approach parallels the Anthropic decomposition, and their finding about prompt emotional charge is relevant to understanding why some topics/framings in the Paired Prompts eval show more uneven treatment.

### Reference Anchor Pairs (Bang et al., ACL 2024)
- **Paper**: [Measuring Political Bias in Large Language Models: What Is Said and How It Is Said](https://aclanthology.org/2024.acl-long.600/)

Prompts LLMs to generate news headlines, then obtains reference anchor distributions by prompting models to generate opposed-stance responses. Decomposes bias into **content** (substance) and **style** (lexical polarity). Found LLMs show different political views depending on the topic — more liberal on reproductive rights, more conservative on immigration.

**Relevance**: The content/style decomposition is analogous to what Anthropic does with even-handedness (behavioral parity) vs. opposing perspectives (content of hedging). The topic-dependent directionality finding motivates our per-category differential analysis.

---

## 4. Evidence for Left-Right Asymmetry

### Stanford User-Perception Study (May 2025)
- **Paper**: [Measuring Perceived Slant in Large Language Models Through User Evaluations](https://www.gsb.stanford.edu/faculty-research/working-papers/measuring-perceived-slant-large-language-models-through-user)
- **Stanford Report**: [Study finds perceived political bias in popular AI models](https://news.stanford.edu/stories/2025/05/ai-models-llms-chatgpt-claude-gemini-partisan-bias-research-study)

180,126 assessments from 10,007 U.S. respondents across 30 topics and 24 LLMs. Found that for 18/30 questions, nearly all LLMs were perceived as left-leaning — by both Republicans and Democrats. OpenAI models had the most perceived left-leaning slant (4x greater than Google). Google and DeepSeek were statistically indistinguishable from neutral.

**Relevance**: Strong evidence for the left-right asymmetry our differential metrics (Section 4) aim to quantify. Notably, this is *perceived* bias from users, while we measure *behavioral* asymmetry through the evaluation rubric — the two may or may not align.

### Manhattan Institute Integrative Approach (Jan 2025)
- **Report**: [Measuring Political Preferences in AI Systems](https://manhattan.institute/article/measuring-political-preferences-in-ai-systems-an-integrative-approach)

Used Congressional Record partisan bigrams as reference. Found that conversational LLMs generate text with more positive sentiment toward left-of-center public figures, with greater variance in sentiment toward right-of-center figures. Democratic partisan terms are more frequent than Republican partisan terms in LLM outputs.

**Relevance**: Provides an alternative methodology for detecting directional bias using lexical markers rather than judge-based evaluation, which could serve as an independent validation of our differential metrics.

### Promptfoo Independent Evaluation (July 2025)
- **Blog**: [Evaluating political bias in LLMs](https://www.promptfoo.dev/blog/grok-4-political-bias/)

2,500 questions across models. Found all popular AIs are left of center, with Claude Opus 4 and Grok closest to neutral. Grok is "politically bipolar" with a 67.9% extremism rate — wild swings between far-left and far-right positions. GPT-4.1 is the most left-leaning in both responses and judgments. Uses both direct bias scoring (7-point Likert) and cross-model scoring.

**Relevance**: Their cross-model scoring methodology (having each model judge other models) is directly relevant to our multi-judge dynamics analysis. The finding about Grok's bipolarity is interesting for our differential metrics — aggregate scores can mask opposing extremes.

### Large Means Left (May 2025)
- **Paper**: [Large Means Left: Political Bias in Large Language Models Increases with Their Number of Parameters](https://arxiv.org/html/2505.04393v1)

Confirms left-leaning bias severity increases with model size and varies by prompt language.

**Relevance**: If bias scales with model size, this has implications for which models are used as judges — larger judge models may themselves have more political bias, creating a confound in the evaluation.

### Beyond Partisan Leaning (Dec 2024, updated May 2025)
- **Paper**: [Beyond Partisan Leaning: A Comparative Analysis of Political Bias in Large Language Models](https://arxiv.org/abs/2412.16746)

Introduces a two-dimensional framework: partisan orientation on polarized topics and sociopolitical engagement on less polarized issues. Finds most models lean center-left. Model scale and openness are not strong predictors — alignment strategy and institutional context matter more.

**Relevance**: Suggests the differential bias we observe may be a function of alignment training rather than model architecture, which connects to our system prompt sensitivity experiments.

---

## 5. LLM Judge Dynamics and Position Bias

### Judging the Judges: Position Bias (Shi et al., IJCNLP-AACL 2025)
- **Paper**: [arXiv:2406.07791](https://arxiv.org/abs/2406.07791)
- **ACL Anthology**: [aclanthology.org/2025.ijcnlp-long.18](https://aclanthology.org/2025.ijcnlp-long.18/)

Systematic study of position bias across 15 LLM judges, 22 tasks, ~40 solution-generating models, and 150,000+ evaluation instances. Introduces three metrics: **repetition stability**, **position consistency**, and **preference fairness**. Confirms position bias is not random chance, varies significantly across judges and tasks, and is strongly affected by the quality gap between solutions.

**Relevance**: Directly motivates our position bias measurement (Section 3.3.1). Their metrics (especially position consistency) can be adopted for the political even-handedness setting. Their finding that bias depends on quality gap is relevant — in political prompts, the "quality gap" may be confounded with political lean.

### Justice or Prejudice? Quantifying Biases in LLM-as-a-Judge (ICLR 2025)
- **Paper**: [OpenReview](https://openreview.net/forum?id=3GTtZFiajM)
- **Project page**: [llm-judge-bias.github.io](https://llm-judge-bias.github.io/)

Identifies 12 key potential biases and proposes the CALM framework for automated bias quantification using principle-guided modification. Found that while advanced models achieve commendable overall performance, significant biases persist in specific tasks.

**Relevance**: CALM's systematic bias quantification could be applied to the political evaluation setting. Their 12 bias types provide a checklist for our multi-judge analysis.

### CyclicJudge: Mitigating Judge Bias (March 2026)
- **Paper**: [arXiv:2603.01865](https://arxiv.org/html/2603.01865)

Proves a variance decomposition for judge bias in three steps (grand mean, law of total variance, finite population correction). References work showing ANOVA and ICC analyses reveal rank inversions in single-run leaderboards.

**Relevance**: Variance decomposition methodology is directly applicable to our verdict stability analysis (Section 3.3.2). Could help disentangle judge variance from true model differences.

### Judge Reliability Harness (Chehbouni et al., March 2026)
- **Paper**: [arXiv:2603.05399](https://arxiv.org/html/2603.05399)

Stress-tests the reliability of LLM judges. Argues that enthusiasm for LLM judges has outpaced rigorous scrutiny of their validity and reliability.

**Relevance**: Directly motivates our entire Section 3. Provides a framework for the kind of reliability testing we propose.

### Multi-Judge Panels: Practical Findings
- **Source**: [LLM as a Judge: A 2026 Guide](https://labelyourdata.com/articles/llm-as-a-judge)

Running 3–5 models with majority vote reduces biases by 30–40% but costs 3–5x more. Position bias shows up to 40% GPT-4 inconsistency when orderings are swapped. Recommended mitigation: evaluate both (A,B) and (B,A) orderings and only count consistent wins.

**Relevance**: Practical guidance for our position-flipping protocol. The 30–40% bias reduction from multi-judge panels motivates the Language Model Council approach.

---

## 6. Priming, Contamination, and Subjectivity in LLM Judges

### Humans or LLMs as the Judge? A Study on Judgement Bias (EMNLP 2024)
- **Paper**: [aclanthology.org/2024.emnlp-main.474](https://aclanthology.org/2024.emnlp-main.474/)

Compares human and LLM judgment biases. Finds that LLM judges exhibit biases including positional preferences that can be exploited by altering context. The judgments of candidate responses can be "easily hacked by altering their position in the context."

**Relevance**: Motivates our concern about prompt-level priming in direct bias assessment (Section 5). If judges can be manipulated by context, then presenting a politically charged prompt alongside a response may prime the judge's bias assessment.

### Threshold Priming in LLM Relevance Judgments (Dec 2025)
- **Paper**: [Mitigating the Threshold Priming Effect in LLM-Based Relevance Judgments via Personality Infusing](https://arxiv.org/html/2512.00390)

Investigates how exposure to prior stimuli unconsciously influences subsequent LLM judgments (the priming effect). When assessors evaluate documents continuously, previously assessed quality acts as a threshold affecting subsequent judgments. Proposes personality prompting (e.g., High Openness, Low Neuroticism) to mitigate priming.

**Relevance**: Directly relevant to our hypothesis that direct political bias assessment is susceptible to priming (Section 5.2.1). The prompt-response mismatch experiment we propose is essentially a test of whether the prompt acts as a "threshold" that primes the judge's neutrality assessment.

### Perceived Political Bias Reduces Persuasive Abilities (Feb 2026)
- **Paper**: [arXiv:2602.18092](https://arxiv.org/html/2602.18092)

Preregistered experiment (N=2,144): a short message indicating that an LLM was biased against the respondent's party attenuated persuasion by 28%.

**Relevance**: Shows that *perception* of political bias — not just actual bias — has real consequences. This further motivates why evaluation methodology matters: if models are perceived as biased (even if aggregate scores look good), this undermines trust and utility.

---

## 7. Search-Augmented Generation and Political Bias

### Unveiling Political Bias in LLM-Generated Citations (EMNLP 2025)
- **Paper**: [aclanthology.org/2025.emnlp-main.872](https://aclanthology.org/2025.emnlp-main.872.pdf)

Examines political bias in how LLMs select and cite sources in RAG-like settings. Found that Google Search outcomes for political queries show "a slight preference for left-leaning media sources," and that biased queries affect LLM citation behavior — source still matters more than content.

**Relevance**: Directly motivates our search-augmented evaluation (Section 2.4). If search retrieval itself is politically skewed, then grounding responses in search results may not reduce political bias and could even amplify it through selective citation.

### Search Grounding Increases Stereotypical Judgments (Feb 2025)
- **Paper**: [Towards Geo-Culturally Grounded LLM Generations](https://arxiv.org/html/2502.13497)

Found that search grounding significantly improves factual performance but also increases the risk of stereotypical judgments by language models.

**Relevance**: Suggests a trade-off: search grounding may improve factual accuracy in political responses but could introduce new forms of bias through the retrieved content.

---

## 8. Political Bias Mitigation via Activation Steering

### Bias Beyond Borders (Jan 2026)
- **Paper**: [arXiv:2601.23001](https://arxiv.org/html/2601.23001)

Unified framework for mitigating political bias through activation-level steering using contrastive pairs derived from the Political Compass Test. Evaluates on a 50-country PCT benchmark across economic and social dimensions. Finds multilingual political bias is both axis-specific and representation-misaligned across languages.

**Relevance**: Represents an alternative approach to the system-prompt steering we explore. If activation steering can modulate political bias, it may interact with the evaluation metrics in interesting ways.

### Steering Towards Fairness (Aug 2025)
- **Paper**: [arXiv:2508.08846](https://arxiv.org/html/2508.08846v2)

Modular pipeline for political debiasing using contrastive prompting and steering vector interventions. Demonstrates that ensemble-based interventions reduce bias while maintaining fluency.

**Relevance**: Another debiasing technique that could be evaluated with the Paired Prompts benchmark, potentially as a comparison condition in our system prompt sensitivity experiments.

### Feature Steering for Bias Mitigation (Anthropic Research)
- **Paper**: [Evaluating feature steering](https://www.anthropic.com/research/evaluating-feature-steering)

Anthropic found that positively steering "Neutrality and Impartiality" and "Multiple Perspectives" features using sparse autoencoders tends to consistently reduce bias scores across all nine BBQ benchmark dimensions within the feature steering sweet spot.

**Relevance**: Suggests that there are identifiable internal representations of neutrality that can be amplified, which connects to the question of whether models have a latent notion of "political neutrality" that direct assessment tries (perhaps unreliably) to elicit.

---

## 9. General LLM Political Bias Landscape

### German Wahl-O-Mat Case Study (2025)
- **Paper**: [Assessing political bias in large language models](https://link.springer.com/article/10.1007/s42001-025-00376-w)

Used Germany's voting advice application to assess LLM political alignment. Found consistent left-leaning tendency, with larger models (Llama3-70B) aligning more closely with left-leaning parties. Minimal alignment with far-right positions regardless of prompt language.

### LLMs Have Offsetting Extreme Views (May 2025)
- **Paper**: [Large Language Models are often politically extreme, usually ideologically inconsistent, and persuasive](https://arxiv.org/html/2505.04171v1)

Compared 31 LLMs to legislators, judges, and voters. Found that "LLMs' apparently small overall partisan preference is the net result of offsetting extreme views on specific topics." In a randomized experiment, voters discussing political issues with an LLM chatbot were up to 5 percentage points more likely to express the same preferences.

**Relevance**: Critical for interpreting aggregate even-handedness scores. A model can score 94% overall while being extremely uneven on specific topics — which is exactly what differential metrics would reveal.

### LLMs as Western Media Bias Detectors (Jan 2026)
- **Paper**: [An evaluation of LLMs for political bias in Western media](https://arxiv.org/html/2601.06132v1)

Assessed how political leaning within the same outlet shifts before and after geopolitical events. Models produce "model-dependent proxies rather than a gold-standard ground truth."

**Relevance**: Reinforces that LLM political judgments are model-dependent — directly supporting our multi-judge analysis approach.

---

## Summary: Gaps Our Research Fills

| Gap | Prior work status | Our contribution |
|-----|-------------------|------------------|
| Position bias in political pairwise evaluation | Studied in general (Shi et al.) but **not applied to political eval** | First systematic position bias measurement on Paired Prompts |
| Multi-judge dynamics for political eval | Anthropic validated with 2 additional judges on subsample only | Full multi-judge analysis with 6+ judges across all metrics |
| Differential metrics by prompt lean | **Not reported anywhere** for the Paired Prompts benchmark | First decomposition showing left/right asymmetry in refusals and opposing perspectives |
| Verdict stability under repeated trials | Studied generally but **not for political evaluation** | First stability analysis in political context |
| Direct vs. decomposed bias assessment | ICML position paper argues theoretically; **no empirical comparison** | First head-to-head empirical test with stability and mismatch experiments |
| System prompt sensitivity for even-handedness | PolBiX found explicit objectivity didn't help; **not tested on Paired Prompts** | Systematic system prompt ablation on the benchmark |
| Search-augmented political evaluation | EMNLP 2025 studied citation bias; **not applied to even-handedness** | First test of search grounding on political even-handedness metrics |
| Granularity spectrum of decomposition | ICML paper discusses approximation levels theoretically | Empirical test: coarse (direct) vs. current (3-metric) vs. finer decomposition |
