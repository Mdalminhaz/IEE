# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** [Md Minhaz]
**Date:** May 19, 2026
**Capstone Project:** Cybersecurity Threat Detection
**My Component:** Phishing Detection / Email Threat Classification

---

## Part A: Teachable Machine Training

### Training Setup

- **Task:** Phishing vs Legitimate email screenshot classification
- **Training images per class:** 22
- **Test images per class:** 5
- **Total training time:** ~40 seconds

### Test Results

| # | File Name | Actual Class | Predicted Class | Confidence | Correct? |
|---|-----------|--------------|-----------------|------------|----------|
| 1 | test_phish_01.png | Phishing | Phishing | 94% | ✓ |
| 2 | test_phish_02.png | Phishing | Phishing | 88% | ✓ |
| 3 | test_phish_03.png | Phishing | Phishing | 76% | ✓ |
| 4 | test_phish_04.png | Phishing | Phishing | 91% | ✓ |
| 5 | test_phish_05.png | Phishing | Legitimate | 61% | ✗ |
| 6 | test_legit_01.png | Legitimate | Legitimate | 97% | ✓ |
| 7 | test_legit_02.png | Legitimate | Legitimate | 83% | ✓ |
| 8 | test_legit_03.png | Legitimate | Legitimate | 79% | ✓ |
| 9 | test_legit_04.png | Legitimate | Legitimate | 88% | ✓ |
| 10 | test_legit_05.png | Legitimate | Legitimate | 92% | ✓ |

### Confusion Matrix

|  | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 4 | FN = 1 |
| **Actual: Legitimate** | FP = 0 | TN = 5 |

### Calculated Metrics

- **Accuracy:** (4 + 5) / 10 = **90%**
- **Precision:** 4 / (4 + 0) = **100%**
- **Recall:** 4 / (4 + 1) = **80%**
- **F1 Score:** 2 × (1.00 × 0.80) / (1.00 + 0.80) = **88.9%**

### Interpretation

The model achieved 90% accuracy on the held-out test set, with perfect precision but slightly lower recall. The one misclassification was a phishing screenshot that used a minimalist layout closely resembling a legitimate transactional email, which likely fell outside the visual patterns the model learned from its 22 training examples. Recall (80%) is the more important metric here — a missed phishing email reaching a user is far more dangerous than a false alarm — so expanding the training set with more visually subtle phishing examples would be the most impactful improvement. Increasing training images to 40–50 per class with greater stylistic variety would likely push recall above 90%.

---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested

1. **Generic:** `distilbert-base-uncased-finetuned-sst-2-english` — general sentiment (positive/negative)
2. **Fine-Tuned A:** `mrm8488/bert-tiny-finetuned-sms-spam-detection` — spam vs ham classification
3. **Fine-Tuned B:** `cardiffnlp/twitter-roberta-base-sentiment-latest` — 3-class sentiment (negative/neutral/positive)

### Test Inputs

1. Unauthorized login from IP 198.51.100.4 traced to Moscow targeting user John Miller at 3:47 AM
2. Routine firewall rule update completed on fw-01 during scheduled maintenance window
3. Phishing email with spoofed Amazon domain detected targeting finance@acmecorp.com
4. Multiple failed SSH attempts from Beijing IP on Cloudflare production server — 47 attempts in 5 minutes
5. System resource utilization normal across all monitored hosts — no anomalies detected

### Results

| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Label (Score) | Best Model |
|-------|-----------------------|----------------------------|----------------------------|------------|
| Record 1 — Unauthorized login | NEGATIVE (0.9987) | spam (0.8921) | negative (0.9412) | Fine-Tuned A |
| Record 2 — Firewall update | POSITIVE (0.9876) | ham (0.9654) | neutral (0.8733) | Fine-Tuned B |
| Record 3 — Phishing email | NEGATIVE (0.9731) | spam (0.9887) | negative (0.9201) | Fine-Tuned A |
| Record 4 — SSH brute force | NEGATIVE (0.9944) | spam (0.9342) | negative (0.9654) | Fine-Tuned A |
| Record 5 — Normal utilization | POSITIVE (0.9811) | ham (0.9521) | neutral (0.9102) | Fine-Tuned B |

### Analysis

**Generic model strengths:** The generic distilbert sentiment model correctly identified the emotional tone of high-threat records 1, 3, and 4 as negative, and routine records 2 and 5 as positive. For a completely untargeted model it performed surprisingly well at separating the two groups by tone alone.

**Generic model weaknesses:** The generic model has no concept of "threat" — it is simply detecting sentiment. A confident, well-written phishing email that uses neutral or positive language would likely be labeled POSITIVE, completely missing the threat. It also offers no domain-relevant labels; "NEGATIVE" is not actionable in a security context the way "spam" or "phishing" would be.

**Fine-tuned model advantage:** The spam detection model (Fine-Tuned A) consistently flagged all three threat records as "spam" with high confidence (0.89–0.99) and correctly classified both routine records as "ham." Its labels are far more meaningful for a security workflow, and it showed the highest confidence on the clearest phishing case (Record 3, 0.9887).

**Biggest surprise:** The generic sentiment model labeled the routine firewall update (Record 2) as strongly POSITIVE (0.9876) — technically correct in sentiment terms but meaningless for threat detection. More notably, it labeled the SSH brute force attack (Record 4) as NEGATIVE with very high confidence, suggesting some alignment between negative sentiment and threatening language even without any security-specific training.

### Recommended Model for My Capstone Component

**Component:** Phishing / email threat detection
**Primary model:** `mrm8488/bert-tiny-finetuned-sms-spam-detection`
**Why:** This model was explicitly trained to distinguish suspicious/unsolicited content from legitimate messages, which maps directly to phishing vs legitimate classification. Its output labels (spam/ham) are immediately actionable in a security pipeline, and it consistently outperformed both alternatives on threat records with confidence scores above 0.89.

**Confidence threshold:** 0.85 — records scoring above this would be automatically quarantined; records below would be routed to an analyst review queue.

**Priority metric:** **Recall** — in a cybersecurity context, a false negative (missed phishing email) poses direct risk to users and the organization. A false positive (legitimate email flagged for review) is a minor operational inconvenience. Maximizing recall ensures the fewest real threats slip through, even at the cost of some additional analyst workload.

---

## Limitations & Next Steps

With only 10 Teachable Machine test images, the calculated metrics are rough estimates rather than statistically reliable measurements — a production evaluation would require 500+ labeled examples and a proper stratified validation split. The spam detection model was originally trained on SMS messages rather than cybersecurity incident logs, meaning its vocabulary may not fully generalize to network security language; a model fine-tuned specifically on phishing email datasets would likely yield stronger results. Next steps would include testing with the full 20-record cybersecurity dataset from Appendix A, exploring phishing-specific models on Hugging Face, and implementing the confidence-based routing workflow from Bonus Challenge 2 to automate triage between auto-block and human review queues.
