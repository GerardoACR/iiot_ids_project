# iiot_ids_project

An experimental framework for evaluating Machine Learning intrusion detection models on Industrial IoT (IIoT) environments using the ToN_IoT Modbus dataset.  
This repository is part of my research project **“Cyber Threats in IIoT: Can ML Mitigate Vulnerabilities and Prevent Harm?”** for the *Governance della Cybersecurity* course.

---

## 1. Overview

The goal of this repository is to study how classic ML models behave as **intrusion detection systems (IDS)** in a Modbus-based IIoT setting.

The experiments are organised in **two phases**:

- **Phase 1 – “Lab” setting (balanced subset)**  
  Models are trained and tuned on the `Train_Test_IoT_Modbus` subset, where classes are much more balanced.  
  This phase is meant to:
  - explore baseline performance for binary and multi-class tasks;
  - tune hyperparameters with cross-validation;
  - inspect class-wise errors (false negatives / false positives) under relatively “comfortable” conditions.

- **Phase 2 – “Realistic” setting (full dataset)**  
  The best models from Phase 1 are retrained and evaluated on the **full `IoT_Modbus` processed dataset**, which is highly imbalanced and closer to realistic attack frequencies.  
  This phase answers the question: *how well do these models generalise when normal traffic dominates and rare attacks are under-represented?*

The focus is not just on accuracy but on **risk-oriented metrics**, especially:

- **Macro-F1** (treating all classes equally),
- **Class-wise False Negative Rate (missed attacks)**,
- **Class-wise False Positive Rate (false alarms)**.

---

## 2. Repository structure

- `iiot_ids_phase1.ipynb`  
  Phase 1 notebook: balanced Modbus subset (`Train_Test_IoT_Modbus`).
  - Data loading and exploratory analysis.
  - Train/validation/test split with stratification.
  - Feature scaling with `StandardScaler`.
  - Model training and tuning:
    - `DecisionTreeClassifier`
    - `RandomForestClassifier`
    - (SVM with RBF kernel was tested but discarded due to poor performance.)
  - Detailed validation and final test evaluation (F1, FNR, FPR).

- `iiot_ids_phase2.ipynb`  
  Phase 2 notebook: full processed dataset (`IoT_Modbus`).
  - Loading and inspecting the full telemetry.
  - Reusing the best hyperparameters from Phase 1.
  - Training on a stratified train/test split that preserves the real imbalance.
  - Final evaluation of the selected models on both:
    - **binary task**: normal vs. attack;
    - **multi-class task**: normal, backdoor, password, injection, scanning, xss.

- `.vscode/`  
  My editor settings for VS Code.

- `LICENSE`  
  MIT license.

- `README.md`  
  This file.
  
- `CyberThreatsInIIoT.pdf`
  My paper.

- `CyberThreatsInIIoT.pdf`
My ppt.

---

## 3. Dataset

This project uses the **ToN_IoT** datasets released by UNSW Canberra.  
In particular, it works with the **Modbus telemetry**:

- `Train_Test_IoT_Modbus.csv` – balanced subset used in Phase 1.
- `IoT_Modbus.csv` (processed) – full Modbus dataset used in Phase 2.

The datasets are **not included** in this repository.  
They can be downloaded from the official ToN_IoT resources provided by UNSW Canberra.  
Once downloaded, update the `data_path` variables inside the notebooks so that they point to your local files.

---

## 4. Experimental pipeline

### 4.1 Tasks

Both phases consider two complementary learning tasks:

1. **Binary classification**  
   - Target: `label`  
   - `0 = normal`, `1 = attack`.

2. **Multi-class classification**  
   - Target: `type`  
   - Classes: `normal`, `backdoor`, `password`, `injection`, `scanning`, `xss`.

This allows us to analyse not only *whether* an attack is detected, but also *which* type of malicious behaviour is suspected.

### 4.2 Models

The following ML models are implemented:

- **Decision Tree (DT)**
- **Random Forest (RF)**

Support Vector Machines with RBF kernel were also explored, but they consistently under-performed compared to tree-based models on this dataset, especially for minority attack classes, and were therefore excluded from the final comparison.

### 4.3 Training procedure

Common steps across both phases:

1. **Stratified train/validation/test splitting**  
   - Phase 1: 60% train, 20% validation, 20% test on `Train_Test_IoT_Modbus`.  
   - Phase 2: 80% train, 20% test on `IoT_Modbus`, preserving the real class imbalance.

2. **Feature preprocessing**  
   - Four Modbus counters (`FC1`–`FC4`), all integer counts.  
   - Standardisation (`StandardScaler`) fitted **only on the training split**, then applied to validation and test.

3. **Hyperparameter tuning**  
   - `StratifiedKFold` with 5 folds on the training split.  
   - Greedy search over reasonable grids (depth, number of trees, min samples per leaf, etc.).  
   - Main selection criterion: **macro-F1**.

4. **Evaluation**  
   - `classification_report` per class.  
   - Macro-F1 and weighted-F1.  
   - Per-class **False Negative Rate (FNR)** and **False Positive Rate (FPR)** to connect results with risk management (missed attacks vs. false alarms).

---

## 5. Main results

### Phase 1 – Balanced subset (`Train_Test_IoT_Modbus`)

- Both **Random Forest** and **Decision Tree** achieve very strong performance on the binary task  
  (macro-F1 ≈ 0.98 on the test split).
- On the multi-class task, macro-F1 is lower (≈ 0.93 for RF, ≈ 0.92 for DT) but still solid.  
  Minority classes such as `scanning` and `xss` are the hardest to detect: they show higher FNR despite good overall accuracy.

### Phase 2 – Full dataset (`IoT_Modbus`)

- On the **binary task (normal vs. attack)**, Random Forest keeps a high macro-F1 (≈ 0.95) and accuracy (~0.97) despite the strong class imbalance.
- On the **multi-class task**, Random Forest reaches:
  - macro-F1 ≈ 0.90
  - weighted-F1 ≈ 0.97
- As in Phase 1, metrics are excellent for `normal` and frequent attacks (`backdoor`, `injection`), while recall remains more modest for rare but potentially dangerous classes such as `password`, `scanning`, and `xss`.

This confirms that **tree-based models can be very effective on tabular IIoT telemetry**, but class imbalance and minority threats remain critical challenges from a security-governance perspective.
