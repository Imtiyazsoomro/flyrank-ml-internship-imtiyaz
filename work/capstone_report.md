# Capstone Report — AI-Powered Content Refresh Prioritization

*   **Author:** Imtiyaz Soomro
*   **Lane:** Machine Learning
*   **Repo:** https://github.com/imtiyazsoomro/flyrank-ml-internship-imtiyaz
*   **Date:** August 2026

## 0. Abstract
This paper investigates whether historical search-performance signals can accurately predict future content decay to prioritize editorial refreshes. Utilizing a 30,000-row anonymized sample from the FlyRank Search Intelligence Dataset, we analyzed visibility, engagement, and content age metrics over 90-day performance windows. We compared a 180-day staleness heuristic against a Random Forest classifier, employing a strict client-grouped validation split (`GroupShuffleSplit`) to prevent domain-level data leakage. The Random Forest model achieved a 68.6% accuracy and 67.3% precision, significantly outperforming the 45.6% accuracy of the baseline heuristic on the identical holdout set. Ultimately, this model powers a ranked Action Playbook that provides content teams with explicit, human-in-the-loop decision support to efficiently triage high-value decaying pages.

## 1. Problem framing
Digital publishers face the persistent challenge of "content decay," where evergreen pages slowly lose organic search visibility over time. The unit of analysis for this study is the individual webpage. The model outputs a continuous probability score of decay, which is thresholded into categorical action tiers. The cost of a wrong call is substantial: false positives waste expensive editorial hours updating healthy pages, while false negatives result in irrecoverable traffic loss. Machine learning is strictly necessary here because content decay is non-linear; simple age-based heuristics fail to capture the multi-dimensional interactions between impression momentum, ranking volatility, and content staleness.

## 2. Data safety
This project utilized the anonymized starter slice of the FlyRank warehouse. Used features included aggregated metrics such as `impressions_90d`, `avg_position`, and `content_age_days`. To ensure strict public safety and avoid proprietary leakage, all private client identifiers—including raw URLs, domains, and queries—were excluded. Leakage risks were heavily mitigated: `trend_direction` and `trend_pct` were dropped prior to training to prevent target leakage, and the pseudonymous `client_hash_id` was used strictly for grouping the validation split, never as a predictive feature. I confirm no client-identifying information appears anywhere in the `work/` directory.

## 3. Baseline
The baseline established was a simple heuristic rule: flag any webpage that has not been updated in over 180 days as "declining." This represents a fair comparison as it mirrors the standard, time-based content audit practices used by many SEO teams. Evaluated on the strict client-grouped holdout split, this baseline achieved an accuracy of only 45.6%, demonstrating that raw content age is an insufficient proxy for actual performance decay.

## 4. Model / analysis
We selected a Random Forest Classifier, an ensemble tree-based method well-suited for this lane due to its ability to map non-linear feature interactions without requiring aggressive scaling or transformation. The exact feature list included `impressions_90d`, `content_age_days`, `avg_position`, `word_count`, and `days_since_last_update`. We deliberately excluded engagement signals that act as lagging indicators of decay. The target was defined as a binary `is_declining_label`, representing a negative traffic trajectory in the most recent 30-day window compared to the prior 90-day baseline.

## 5. Evaluation
The dataset was split using `GroupShuffleSplit` on `client_hash_id`. This is critical: a random split would allow the model to learn client-specific domain patterns (data leakage), so grouping ensures the model is evaluated on entirely unseen client portfolios.
*   **Baseline Accuracy:** 45.6%
*   **Random Forest Accuracy:** 68.6%
*   **Random Forest Precision:** 67.3%

**Error Analysis:** False positives frequently occurred on high-traffic, evergreen pages. This "Variance Trap" happens because massive-volume pages experience larger absolute daily traffic swings, which the model occasionally misclassifies as symptomatic decline rather than normal seasonality.

## 6. Interpretation
Feature importance extraction revealed that visibility and age metrics heavily outweighed sheer content length. `impressions_90d` (34%), `content_age_days` (23%), and `avg_position` (22%) drove the majority of the model's decisions, while `word_count` (12%) contributed marginally. This is a crucial negative result, debunking the common industry myth that longer word counts inherently protect against content decay.

## 7. Recommendation
The model's outputs support an Editorial Action Playbook, segmented into ranked actions:
1.  **URGENT REFRESH:** High Risk (>60%) + High Value (>1000 impressions). Send immediately to editors for intent-matching updates.
2.  **STANDARD REFRESH:** High Risk (>60%) + Low Value. Batch for bulk updates.
3.  **MONITOR:** Low risk. Preserve editorial budget.
4.  **NO TOUCH:** Pages under 30 days old. Programmatically excluded to prevent early-stage volatility from triggering false alerts.

This provides directional decision-support. We claim confidence in the model's ability to prioritize editorial work, but limit causal claims: the model flags decline probability, but does not guarantee a refresh will reverse it.

## 8. Reproducibility
All executions were run using Python 3, Pandas, and Scikit-Learn. To ensure perfect reproducibility, a random seed (`random_state=42`) was fixed across all splits and algorithms. The complete, end-to-end executable pipeline—from data ingestion to model evaluation—is available in `work/notebooks/capstone.ipynb`.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset (https://flyrank.ai).
