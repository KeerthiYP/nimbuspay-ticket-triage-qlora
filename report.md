# NimbusPay Support Ticket Triage — Technical Report

## 1. Approach

The objective was to transform customer support tickets into structured JSON with eight fields: category, priority, amount_paise, txn_date, txn_id, channel, language, and needs_human.

I used Qwen2.5-1.5B-Instruct with 4-bit QLoRA fine-tuning. This approach was selected to make instruction fine-tuning feasible on a single NVIDIA Tesla T4 GPU.

Training configuration:
- LoRA rank: 16
- Learning rate: 2e-4
- Epochs: 2
- Trainable parameters: approximately 18.46 million
- Best-model training time: approximately 12.92 minutes

## 2. Data Audit and Cleaning

The training data contained inconsistent labels and formatting. The audit identified 25 malformed or truncated JSON labels, 70 valid labels using alternate priority representations, and 30 date-format inconsistencies.

The cleaning process excluded malformed records and normalized supported priority and date representations. The regenerated cleaned dataset contained 1,420 parseable records.

The cleaning log and audit notes are included in the repository. The selected final model came from the raw-data training experiment rather than a retraining run on the regenerated cleaned dataset.

## 3. Experiments and Evaluation

Evaluation used 200 development examples.

| Model / Experiment | Exact Match | Mean Field Accuracy | JSON Validity |
|---|---:|---:|---:|
| Zero-shot Qwen2.5 baselines | 0% | — | 0–16% |
| QLoRA on raw training data | **57.00%** | **92.56%** | **100%** |
| QLoRA cleaned-data experiment | 52.00% | 92.38% | 100% |

The raw-data QLoRA adapter was selected because it achieved the highest observed development exact-match accuracy before postprocessing.

The controlled experiment coverage was limited; these results should not be interpreted as a comprehensive hyperparameter search.

## 4. Failure Analysis

Before postprocessing, priority prediction was the weakest field, achieving 61.5% accuracy. Among 200 development tickets, 77 priority predictions were incorrect.

Notable error patterns included:
- 25 tickets labeled P1 were predicted as P3.
- 37 tickets labeled P2 were predicted as P3.
- Some tickets containing legal escalation language were assigned insufficient urgency.
- Some payment or refund tickets were assigned incorrect priority despite transaction amounts being extracted correctly.

This suggests that the model learned structured extraction more reliably than the conditional business rules governing urgency.

Before postprocessing, needs-human accuracy was 87.5%, indicating further room for improvement in escalation decisions.

## 5. Final Predictions

The selected raw-data QLoRA adapter generated predictions for 400 hidden-test inputs using greedy decoding, followed by rule-based priority postprocessing.

The postprocessing rules prioritize fraud, legal escalation, account-access issues, and high-value refund or failed-payment transactions. Specifically, the priority rules use fraud detection, legal escalation indicators, account-access categories, and transaction amount thresholds to determine urgency.

Four malformed transaction dates were also normalized to YYYY-MM-DD format.

### Development Set Results

| Metric | Before Postprocessing | After Postprocessing |
|---|---:|---:|
| Exact Match Accuracy | 57.00% | **80.00%** |
| Priority Accuracy | 61.50% | **98.50%** |

The postprocessing rules were refined using the same 200-example development set. Therefore, the improved results are not independent validation and may overestimate performance on unseen data.

### Final Test Prediction Validation

- Total prediction records: 400
- Unique ticket IDs: 400
- JSON validity: 100%
- Invalid transaction dates: 0
- Final priority distribution: P1 = 60, P2 = 85, P3 = 255

The final predictions are saved in `predictions_test.jsonl`.

No hidden-test accuracy is claimed because reference labels were unavailable.

## 6. Limitations and Future Improvements

The original model's main limitation was priority classification, particularly its tendency to underpredict urgent cases.

Rule-based postprocessing improved priority accuracy and exact-match accuracy on the development set. However, because the rules were refined on that same set, their generalization to unseen data remains uncertain.

Potential improvements include targeted training examples for legal threats and transaction amount thresholds, additional controlled fine-tuning experiments, and evaluation on a separate held-out validation set.

The submitted results reflect the experiments actually completed, including the observed weaknesses and the subsequent postprocessing improvements.
