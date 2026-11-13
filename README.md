# NimbusPay Support Ticket Triage — QLoRA Fine-Tuning

## Project Overview

This project implements an automated support-ticket triage system for the NimbusPay take-home ML assignment.

The system uses **Qwen2.5-1.5B-Instruct**, fine-tuned using **4-bit QLoRA**, to convert customer support messages into structured JSON containing ticket category, priority, transaction details, language, and human-escalation requirements.

## Model Performance

Evaluation on 200 development examples:

| Metric | Result |
|---|---|
| Exact Match Accuracy | **57.00%** |
| Mean Field Accuracy | **92.56%** |
| JSON Validity | **100.00%** |
| Category Accuracy | 97.50% |
| Priority Accuracy | 61.50% |
| Amount Accuracy | 96.50% |
| Transaction Date Accuracy | 98.00% |
| Transaction ID Accuracy | 100.00% |
| Channel Accuracy | 99.50% |
| Language Accuracy | 100.00% |
| Needs Human Accuracy | 87.50% |

These results are from the development set, not the hidden test set.

## Model and Training

- Base model: `Qwen/Qwen2.5-1.5B-Instruct`
- Method: 4-bit QLoRA with PEFT
- LoRA rank: 16
- Training epochs: 2
- Learning rate: 2e-4
- Trainable parameters: approximately 18.46 million
- Training hardware: NVIDIA Tesla T4
- Best-model training time: approximately 12.92 minutes

The final submission uses the adapter trained on the original training experiment, which achieved better development-set exact-match accuracy than the subsequent cleaned-data experiment.

## Output Schema

The model predicts eight fields:

1. `category`
2. `priority`
3. `amount_paise`
4. `txn_date`
5. `txn_id`
6. `channel`
7. `language`
8. `needs_human`

Predictions follow the JSON schema supplied with the assignment.

## Data Preparation

The data audit identified malformed or truncated labels, inconsistent priority representations, and date-format inconsistencies.

The regenerated cleaned dataset contains 1,420 JSON-parseable records. Cleaning details are documented in `cleaning_log.csv` and `data_audit.md`.

The final selected adapter was trained in the raw-data experiment; the regenerated cleaned dataset is included for reproducibility and auditing.

## Experiments

| Experiment | Exact Match | Field Accuracy | JSON Validity |
|---|---:|---:|---:|
| Raw-data QLoRA | **57.00%** | **92.56%** | 100% |
| Cleaned-data QLoRA | 52.00% | 92.38% | 100% |

Zero-shot baseline experiments were also performed with Qwen2.5 models. Their exact-match accuracy was 0% under the evaluated prompting configurations.

## Error Analysis

Priority classification was the largest weakness, with 77 incorrect priority predictions among 200 development examples.

Common errors included predicting `P3` for tickets requiring `P1` or `P2`, particularly where escalation depended on transaction amounts or legal-threat language.

Future improvements could include targeted priority examples, stronger rule-aware training, and additional controlled experiments.

## Repository Contents

- `notebook92df7763f0.ipynb` — Kaggle experiment notebook
- `adapter/` — trained QLoRA adapter weights and configuration
- `predictions_test.jsonl` — predictions for 400 hidden-test inputs
- `train_clean.jsonl` — regenerated cleaned training data
- `cleaning_log.csv` — cleaning audit log
- `data_audit.md` — data-quality findings
- `README.md` — project documentation

## Test Predictions

The final pipeline generated predictions for **400 hidden-test inputs** using the QLoRA fine-tuned model followed by rule-based priority postprocessing.

Postprocessing applies priority rules for fraud, legal escalation, account-access issues, and high-value refund or failed-payment transactions. Four malformed transaction dates were also normalized to `YYYY-MM-DD`.

Final validation confirmed:

- 400 prediction records
- 400 unique IDs
- 100% parseable JSON outputs
- 0 invalid transaction dates

On the 200-example development set, the original model achieved **57.00% exact-match accuracy** and **61.50% priority accuracy**. With rule-based priority postprocessing, these increased to **80.00%** and **98.50%**, respectively.

The postprocessing rules were refined using this same development set, so these results are not independent validation. Hidden-test accuracy remains unknown because test labels were not provided.

## Limitations

The model performs well on structured field extraction but is less reliable for priority classification and human-escalation decisions. The limited experiment coverage and observed priority errors should be considered when interpreting the results.
