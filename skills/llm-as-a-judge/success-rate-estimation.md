# Estimating True Success Rates with Imperfect Judges

After finalizing and freezing your LLM-as-Judge prompt, use this procedure to estimate the true pass rate on unlabeled production traces — correcting for the judge's known error rates.

This is not optional. Without bias correction, the raw pass rate from an imperfect judge is misleading. Building trust in the judge's outputs is the foundation for building trust in your product.

---

## The Problem

Your judge is imperfect. If you run it over thousands of new outputs and simply count "Pass" predictions, you get a biased estimate. The judge occasionally misses failures (inflating the pass rate) or flags correct outputs (deflating it).

## The Procedure

### Step 1: Measure Judge Accuracy on Test Set

Compare judge predictions to human labels on the held-out Test set:

```
TPR = |{i : y_i = Pass, ŷ_i = Pass}| / |{i : y_i = Pass}|
TNR = |{i : y_i = Fail, ŷ_i = Fail}| / |{i : y_i = Fail}|
```

- **TPR (True Positive Rate):** Fraction of actual Passes the judge correctly labels Pass.
- **TNR (True Negative Rate):** Fraction of actual Fails the judge correctly labels Fail.

### Step 2: Observe Raw Success Rate

Run the judge on `m` new, unlabeled traces. Let `k` = number labeled "Pass."

```
p_obs = k / m
```

### Step 3: Correct for Bias (Rogan-Gladen Formula)

```
θ̂ = (p_obs + TNR - 1) / (TPR + TNR - 1)    [clipped to 0, 1]
```

If `TPR + TNR - 1 ≈ 0` (e.g., both ~50%), the judge is no better than random chance and the correction is invalid.

### Step 4: Bootstrap Confidence Interval

Quantify uncertainty by bootstrapping over the Test set:

1. For each of B iterations (e.g., B = 20,000):
   - Sample (with replacement) from the Test set's (human_label, judge_prediction) pairs
   - Recompute TPR* and TNR* on that sample
   - Apply the correction: θ* = (p_obs + TNR* - 1) / (TPR* + TNR* - 1)
   - Clip to [0, 1] and record
2. Take the 2.5th and 97.5th percentiles as the 95% confidence interval.

---

## Python Implementation

Uses only numpy. Also available as the open-source `judgy` library: https://github.com/ai-evals-course/judgy

```python
import numpy as np

def estimate_success_rate(
    test_labels,
    test_preds,
    unlabeled_preds,
    B=20000
):
    """
    Estimate the true success rate of an LLM pipeline using
    an imperfect LLM-as-Judge, with bias correction and
    bootstrap confidence intervals.

    Args:
        test_labels:     array of 0/1, human labels on test set (1 = Pass).
        test_preds:      array of 0/1, judge predictions on test set (1 = Pass).
        unlabeled_preds: array of 0/1, judge predictions on unlabeled data (1 = Pass).
        B:               number of bootstrap iterations.

    Returns:
        theta_hat: point estimate of true success rate.
        L, U:      lower and upper bounds of 95% bootstrap CI.
    """
    test_labels = np.asarray(test_labels, dtype=int)
    test_preds = np.asarray(test_preds, dtype=int)
    unlabeled_preds = np.asarray(unlabeled_preds, dtype=int)

    # Step 1: Judge accuracy on test set
    P = test_labels.sum()
    F = len(test_labels) - P
    TPR = ((test_labels == 1) & (test_preds == 1)).sum() / P
    TNR = ((test_labels == 0) & (test_preds == 0)).sum() / F

    # Step 2: Raw observed success rate
    p_obs = unlabeled_preds.sum() / len(unlabeled_preds)

    # Step 3: Correct estimate
    denom = TPR + TNR - 1
    if denom <= 0:
        raise ValueError("Judge accuracy too low for correction")
    theta_hat = np.clip((p_obs + TNR - 1) / denom, 0, 1)

    # Step 4: Bootstrap CI
    N = len(test_labels)
    idx = np.arange(N)
    samples = []
    for _ in range(B):
        boot_idx = np.random.choice(idx, size=N, replace=True)
        lbl_boot = test_labels[boot_idx]
        pred_boot = test_preds[boot_idx]
        P_boot = lbl_boot.sum()
        F_boot = N - P_boot
        if P_boot == 0 or F_boot == 0:
            continue
        TPR_star = ((lbl_boot == 1) & (pred_boot == 1)).sum() / P_boot
        TNR_star = ((lbl_boot == 0) & (pred_boot == 0)).sum() / F_boot
        denom_star = TPR_star + TNR_star - 1
        if denom_star <= 0:
            continue
        theta_star = np.clip((p_obs + TNR_star - 1) / denom_star, 0, 1)
        samples.append(theta_star)

    if not samples:
        raise RuntimeError("No valid bootstrap samples; check inputs")

    L, U = np.percentile(samples, [2.5, 97.5])
    return theta_hat, L, U
```

### Example Usage

```python
# Test set: 50 Pass, 50 Fail traces with human labels
test_labels = [1]*50 + [0]*50   # human ground truth
test_preds  = [1]*45 + [0]*5 + [0]*42 + [1]*8  # judge predictions (TPR=90%, TNR=84%)

# 500 new unlabeled traces judged by the LLM
unlabeled_preds = [1]*440 + [0]*60  # 440 judged Pass out of 500

theta, L, U = estimate_success_rate(test_labels, test_preds, unlabeled_preds)
print(f"Estimated true success rate: {theta:.3f}")
print(f"95% CI: [{L:.3f}, {U:.3f}]")
```

---

## Interpreting Results

- **If the CI is tight and above your SLO:** The pipeline is performing acceptably with high confidence.
- **If the CI is wide:** You need either better judge accuracy (improve TPR/TNR) or more labeled data.
- **If the lower bound is below your threshold:** You cannot guarantee the pipeline meets the bar. Improve the pipeline or the judge, then re-estimate.

## Sensitivity Analysis Summary

- **Improving TPR** narrows the confidence interval the most.
- **Judge errors inflate uncertainty** (wider CIs) rather than shifting the corrected estimate.
- **The bias correction reliably removes systematic error** in both directions (under- and over-counting).
- **More labeled Test data** improves the reliability of TPR/TNR estimates, which tightens everything downstream.

## Two Levers for Narrower CIs

1. **Increase test-set size** — more human-labeled examples tighten TPR/TNR estimates.
2. **Improve judge accuracy** — especially TPR, which has the largest impact on CI width.
