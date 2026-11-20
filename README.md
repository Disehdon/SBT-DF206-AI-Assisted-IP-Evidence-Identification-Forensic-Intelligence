# AI-Assisted IP Evidence Identification & Forensic Intelligence

> **SBT-DF206 — AI for Forensics** · Case Reference: `ICDFA-DFIR-AI-026`  
> Fellowship in Web Application Security & Digital Forensics  
> International Cybersecurity and Digital Forensics Academy (ICDFA)

---

## Overview

This repository contains a reproducible forensic triage lab that evaluates whether an AI-assisted workflow can distinguish **probable IPv4 address evidence** from misleading IP-like text in mobile-device exports.

The study compares two approaches on a fixed holdout set:

1. **Rule-based baseline** — regex extraction + strict `ipaddress.IPv4Address` validation.
2. **Machine-learning classifier** — TF-IDF character n-grams + Logistic Regression.

The central finding is that **machine learning can support analyst triage but must never replace deterministic validation or examiner judgement**.

---

## Objectives

- Preserve and hash evidence for integrity verification.
- Profile the supplied dataset for duplicates, nulls and label quality.
- Extract candidate IPv4 strings using regular expressions.
- Validate candidates with Python's strict IPv4 parser.
- Train and evaluate an offline, reproducible classifier.
- Review historical fine-tuning artefacts for completeness.
- Document disagreements between source labels and technical validity.
- Recommend an operationally safe, human-supervised workflow.

---

## Tools & Technologies Used

### Environment
| Tool | Purpose |
|---|---|
| **Kali Linux** | Operating system for the lab environment |
| **Python 3.10+** | Core programming language |
| **Bash / Terminal** | Command execution and evidence handling |
| **Python `venv`** | Isolated virtual environment |

### Forensic & Data Handling
| Tool | Purpose |
|---|---|
| **`sha256sum`** | Cryptographic hashing of preserved evidence files |
| **`evidence_manifest.txt`** | Recorded SHA-256 hashes for chain of custody |
| **Python `ipaddress`** | Strict IPv4 address validation (`IPv4Address`) |
| **Python `re` (regex)** | Candidate IP pattern extraction |
| **JSONL** | Prepared model input format (one JSON object per line) |

### Data Analysis & Machine Learning
| Tool | Purpose |
|---|---|
| **`pandas`** | CSV profiling, null checks, duplicate detection |
| **`scikit-learn`** | `TfidfVectorizer`, `LogisticRegression`, classification reports |
| **`matplotlib`** | Historical training-loss visualisation |

### Evaluation
| Tool | Purpose |
|---|---|
| **`classification_report`** | Precision, recall, F1-score, support per class |
| **Holdout split (183 / 46)** | Fixed train–test partition for reproducibility |
| **`random_state=42`** | Deterministic model behaviour |

---

**Setup & Reproduction**
1. Clone the repository
bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
2. Create and activate a virtual environment
bash
python3 -m venv venv
source venv/bin/activate
3. Install dependencies
bash
pip install pandas scikit-learn matplotlib
4. Verify evidence integrity
bash
sha256sum originals/* > evidence_manifest.txt
cat evidence_manifest.txt
5. Run the extraction and training pipeline
bash
python code/extract_ips.py
python code/train_model.py
6. Review historical artefacts
bash
python code/historical_metrics.py

---

**Key Results**
Metric	Rule-Based Baseline	ML Classifier
Accuracy	0.74	0.83
Negative precision	0.00	1.00
Negative recall	0.00	0.33
Positive precision	0.74	0.81
Positive recall	1.00	1.00
Positive F1-score	0.85	0.89
Dataset Profile
Source Label	Count
A	177
N	32
B	12
C	4
E	4
Total	229
Duplicates: 77 (≈ 33.6%) — creates data-leakage risk.

Null values: 0 across ip, class and text.

Train/Test split: 183 / 46 records.

Historical Artefacts
Metric	Value
Step	733.0
Accuracy	0.9783 (≈ 97.8%)
Precision	0.9714
Recall	1.0
F1	Missing (classification/f1 column absent)

---

The historical 97.8% accuracy cannot be compared directly with the 83% offline-model accuracy because dataset, split and evaluation conditions were not shown to be equivalent. The legacy ada model and obsolete API endpoints also prevent re-execution.

---

**Notable Findings**
Label noise is present. The string 1.000.000.000 is labelled positive (A) despite being an invalid IPv4 address (octets > 255).

Context matters. 3608.80.24.1.8 contains the valid embedded address 80.24.1.8, but the source label is negative (N). A purely local regex match can produce false positives.

Accuracy is misleading on imbalanced data. The baseline achieved 74% accuracy while scoring 0.00 for negative-class precision and recall.

The ML model improved contextual filtering but still missed ~67% of negative cases (negative recall = 0.33).

---

**Recommendations**
Preserve evidence integrity. Retain originals and SHA-256 hashes; treat any processed file as a traceable derivative.

Use strict validation first. Regex locates candidates; ipaddress.IPv4Address verifies syntax.

Never overwrite source labels. Record technical disagreements in a separate review field.

Resolve duplicates before splitting. Prevent train/test leakage in future experiments.

Document the five-class label mapping (A, N, B, C, E) before training.

Use ML only for analyst triage. Refer ambiguous records to a human examiner.

Report per-class metrics. Include precision, recall, F1 and confusion matrices.

Do not cite the historical 97.8% accuracy as a verified benchmark until the F1 metric and original configuration are recovered.

---

**Final Decision**
Deploy as analyst assistance only.
The recommended workflow is layered and human-supervised: preserve and hash the evidence → extract candidates → validate strictly → apply ML only to ambiguous contexts → require examiner review.

---

**Academic Declaration**
This case study was completed as part of the Fellowship in Web Application Security & Digital Forensics at the International Cybersecurity and Digital Forensics Academy (ICDFA).

The supplied database was used solely for this authorised academic exercise.

No live network, host or account was contacted.

Unrelated personal data shown in source screenshots was redacted.

Screenshots and terminal outputs are retained as supplied so visual evidence is not silently modified.

---

**Author**
Ibrahim Diseh Garba
Student ID: 2025/FWSD/11521
Programme: Fellowship in Web Application Security & Digital Forensics
Batch: BATCH-B2025 · L1/S2
Case Reference: ICDFA-DFIR-AI-026
