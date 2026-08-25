# Capstone Report — <your lane>

- **Author:** Mneeha
- **Lane:** Structured Content Archetype Clustering
- **Repo:** github.com/0mneeha93/ML-Track
- **Date:** 25/08/2026

## 0. Abstract

Can content pages be grouped into stable, honest performance archetypes that tell you what action to take? This study clusters 175,304 visible FlyRank pages (March 2026) into four archetypes using K-Means on position, impressions, CTR, and word count, validated with a same-month check and a stronger out-of-time test (fit on February, applied to March). The clustering recovers and extends a simple baseline rule, concentrating 40.9% of its flagged "improvement candidates" into one high-traffic archetype.

## 1. Problem framing

Content teams routinely face a version of the same question for every page in their portfolio: does this page need protecting, improving, rewriting, merging, pruning, or simply monitoring?
Answering that page-by-page is slow, and answering it with a single flat rule "flag anything with low CTR," for instance misses real structural differences between pages. A high-traffic page underperforming its potential looks nothing like a buried page nobody sees, even if both technically fail the same rule.
This paper builds a clustering model to group pages into a small number of interpretable performance archetypes, then tests two things a reader should be skeptical of by default: whether the clusters are real and stable, not noise dressed up as structure, and whether the clustering actually adds decision-making value beyond a much simpler baseline rule. The decision this work supports is triage where should limited content-team attention go first, and on what basis.

## 2. Data safety

This paper uses the FlyRank ML Internship dataset, a warehouse release of Google Search Console and GA4 performance data hosted on Hugging Face (FlyRank/internship-warehouse), accessed via a read-only DuckDB connection.
Two monthly partitions were used: February 2026, used to fit the clustering model, and March 2026, used as an out-of-time test set to check whether the model's archetypes hold up on data it never saw during fitting. Features were built from fact_content_daily_performance aggregated to the content level: average search position, total impressions, weighted click-through rate joined with dim_content for word count.
Population filter: only pages with a recorded average search position greater than zero were included i.e., pages with at least some search visibility that month. This is a disclosed scoping choice, not an oversight: pages with zero visibility don't carry the position/CTR signal the clustering depends on. As a result, the archetypes described in this paper characterize visible content only, and should not be extended to claims about unindexed or invisible pages.

## 3. Baseline

To check whether clustering adds anything beyond a simple rule, pages were also scored against a hand-written baseline: flagged as an "improvement candidate" if impressions ≥ 500, position ≤ 10, and CTR < 0.3%. Of 175,304 pages, 23,324 (13.3%) were flagged. Flagged pages concentrate heavily in one cluster (see Results) evidence the clustering recovers a version of the same signal the baseline targets, while also separating pages the simple rule can't distinguish.

## 4. Model / analysis

 This is an unsupervised grouping task with no ground-truth label to predict, so K-Means clustering was used rather than a classification method. The number of clusters (k=4) was chosen using silhouette score across k=3–6 (0.447, 0.475, 0.478, 0.482) improvement flattens noticeably after k=4, so 4 was chosen as the simplest cluster count that captures the main structure without over-splitting.

## 5. Evaluation

K-Means has no label to leak, so the relevant validation question is stability: does the clustering structure hold up on data it wasn't fit on? Two checks were run:
Row-resample check:
K-Means fit on a random 70% subset of March 2026 pages produced cluster centers closely matching the full-data centers evidence the clustering isn't an artifact of any particular sample of rows. This is the weaker of the two tests: it never leaves the same time window.
Out-of-time check:
K-Means was fit on February 2026 data only (151,956 pages), then applied without refitting to March 2026 (175,304 pages). The same four archetypes reappeared with broadly consistent profiles, though page counts shifted meaningfully between clusters most notably, the high-traffic/low-position cluster grew from 2,776 pages under the original March-only fit to 23,040 pages under this out-of-time test. This is the more deployment-realistic result, mimicking a model fit on past data and applied to new content.

## 6. Interpretation

Based on March 2026 data across 175,304 pages, four archetypes were identified:Steady Middle: Encompasses 143,105 pages with an average position of 10.1, 1,224 impressions, 0.32% CTR, and a 2,715 word count.High-Traffic Workhorses: Contains 2,776 pages holding an 11.6 position but driving a massive 32,220 impressions, with a 0.29% CTR and a 3,004 word count.Buried and Struggling: Includes 29,029 pages ranking low at a 51.8 position, yielding 536 impressions, a minimal 0.09% CTR, and a 2,840 word count.Suspicious Outliers: Comprises 394 flagged pages showing anomalous metrics, including an 8.6 position, a high 62.5% CTR, only 2 impressions, and a 1,464 word count. Model vs. baseline, same population. The baseline rule (impressions ≥ 500, position ≤ 10, CTR < 0.3%) flagged 23,324 of 175,304 pages (13.3%).

## 7. Recommendation

An action playbook mapping each archetype to a recommended next step, in priority order.

Monitor + test
High-Traffic Workhorses
These pages already carry the portfolio's visibility. The 40.9% baseline-flag rate for low CTR at good position suggests a real optimization opportunity title and snippet testing rather than a "leave alone" stance.

Review, case-by-case
Buried and Struggling
Deep position (avg. 51.8) with meaningful impressions on some pages suggests recoverable content. Pages with minimal impressions in this group are better prune candidates than rewrite candidates.

Periodic check-in
Steady Middle
The largest group by far; treat as the baseline population to re-check each month for pages drifting into "Buried" or emerging as new "Workhorses."

Exclude & investigate
Suspicious Outliers
Not a real content archetype a near-zero-impression measurement artifact. Recommend excluding it from any action list and checking whether these URLs have tracking or indexing issues.

## 8. Reproducibility

Every figure and number in this paper traces back to a committed, runnable notebook.

- [work/notebooks/w05_model.ipynb](https://github.com/0mneeha93/ML-Track/blob/main/work/notebooks/w05_model.ipynb) — modeling & baseline
- [work/notebooks/w06_validation_audit.ipynb](https://github.com/0mneeha93/ML-Track/blob/main/work/notebooks/w06_validation_audit.ipynb) — validation & audit
- [github.com/0mneeha93/ML-Track](https://github.com/0mneeha93/ML-Track) — full repository

## 9. Acknowledgments & data credit

- [Built on the FlyRank ML Internship dataset](https://flyrank.ai/)
---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
