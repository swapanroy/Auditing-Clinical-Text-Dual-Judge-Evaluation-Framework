# Auditing-Clinical-Text-Dual-Judge-Evaluation-Framework


### 1. Judge Agreement (LLM vs. LLM)

Because your sample size reached $n = 100$, the minimum-N gate was cleared, allowing the pipeline to output meaningful agreement labels rather than withholding them:

* **Relevance (`AGREES`, QWK = 0.798, Spearman = 1.000):** Both judges achieved near-perfect consensus on whether the summaries stayed strictly on-task.


* **Safety (`UNCERTAIN`, QWK = 0.797, Spearman = 0.583):** Strong quadratic weighted kappa, though wider confidence intervals (`0.607` to `1.000`) keep it labeled as uncertain.


* **Factuality (`UNCERTAIN`, QWK = 0.477, Spearman = 0.344):** Moderate agreement; factuality checking remains a trickier dimension where cross-model interpretation naturally diverges.


* **Completeness (`DISAGREES`, QWK = -0.027):** Virtually zero/slightly negative agreement. Claude and Gemini evaluated what constitutes "complete" clinical coverage differently on this dataset.


* **Presentation (`NOT MEASURABLE`):** Triggered the constant-score guard because all scores mapped to a single value, making variance unmeasurable.



### 2. Sensor Health & Constant-Score Flags

* **Missing Rate (`0.0`):** Both Claude and Gemini successfully parsed every single response into valid JSON structures without unparseable failures.


* **Constant Scores (`presentation`):** Both Claude and Gemini triggered the **`CHECK: constant scores`** flag for the `presentation` dimension (`distinct_scores = 1`), meaning both models gave every summary the exact same score for formatting/layout. This highlights that structural presentation lacks enough nuance in this prompt configuration to force grading variance.



### 3. Mutation Testing (Error Detection)

Mutation testing injected artificial flaws into summaries to check if the judges could catch them:

* **Off-topic & Unsafe Advice & Unsupported Claims:** Both Claude and Gemini excelled here, achieving **1.000 detect rates** for catching injected off-topic sentences, hazardous guidance, and fake clinical numbers.


* **Disorganized & Dropped Diagnosis:** Gemini struggled more with catching structural disorganization (0.833 detect rate), while Claude successfully caught structural drop-offs and mutations at higher rates than Gemini.



### Key Takeaway

Your end-to-end framework successfully scaled to 100 notes, bypassing the minimum-$n$ gate, and surfaced critical insights: while the two LLMs strongly agree on **relevance** and **safety**, they fundamentally diverge on **completeness**, and both suffer from uniform scoring behavior on **presentation**.
