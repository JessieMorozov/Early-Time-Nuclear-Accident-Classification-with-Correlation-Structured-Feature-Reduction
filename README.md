# Early-Time-Nuclear-Accident-Classification-with-Correlation-Structured-Feature-Reduction

# Early-Time Nuclear Accident Classification with Correlation-Structured Feature Reduction

## Overview

This repository contains a reproducible machine-learning pipeline for early-time nuclear accident classification using the public Nuclear Power Plant Accident Data (NPPAD) dataset.

The project investigates whether accurate accident classification requires the full available instrumentation set, or whether a substantially smaller subset of physically coupled sensor groups can preserve predictive performance.

Using correlation-structured feature clustering and validation-based subset selection, the final pipeline reduces the feature space from 81 cleaned sensor variables to 20 selected variables while maintaining strong held-out classification performance.



## Research Question

Can early nuclear accident classification preserve high predictive accuracy while using a reduced set of physically coupled sensor groups rather than the full instrumentation set?



## Dataset

This work uses the public Nuclear Power Plant Accident Data (NPPAD) dataset:

NuclearPowerPlantAccidentData
https://github.com/thu-inet/NuclearPowerPlantAccidentData

The dataset contains simulated accident scenarios generated for nuclear power plant safety research.

This repository does not contain the dataset itself. The notebook automatically downloads the dataset from the original source.

Several accident categories were excluded to match the experimental setup used in this project:

```python
["ATWS", "LACP", "LOF", "Normal", "SP", "TT"]
```

The remaining accident categories are used for multi-class classification.



## Methodology

### 1. Train / Validation / Test Split

Simulations are split into:

* 60% training
* 20% validation
* 20% testing

All model-selection decisions are performed using training and validation data only.

The test set is reserved exclusively for final evaluation.

### 2. Feature-Space Definition

Common numeric variables are identified using training simulations only.

Dead or unusable instrumentation channels are removed:

```python
["EBK", "MCRT", "MDBR", "MGAS", "QFCL", "QRHR", "TCRT"]
```

### 3. Time-Series Processing

Each simulation is:

* downsampled,
* converted into a fixed-length representation,
* padded when necessary,
* standardized using a scaler fit exclusively on the training split.

### 4. Near-Constant Feature Removal

Low-variance variables are identified using training data only and removed from all splits using the same learned mask

### 5. Correlation-Structured Clustering

Pearson correlation clustering is applied to the cleaned training data

Features with highly similar temporal behavior are grouped into clusters intended to approximate physically coupled instrumentation systems

### 6. Validation-Based Feature Selection

Clusters are ranked using validation performance.
Incremental cluster addition is then performed using validation accuracy and macro-F1

The selected subset is chosen before any test-set evaluation

### 7. Final Classification

Classification is performed using:

* Non-Myopic Early Classification (NMEC)
* Random Forest base classifier

The final model is retrained on the combined training and validation sets before final testing.

---

## Leakage Safeguards

The cleaned pipeline explicitly avoids several common sources of data leakage:

* Common feature definitions learned from training data only
* StandardScaler fit on training data only
* Low-variance feature removal learned from training data only
* Correlation clustering learned from training data only
* Feature-subset selection performed on validation data only
* Test set used exactly once for final evaluation

---

## Results

Final selected feature count:

```text
20 features
```

Held-out test performance:

```text
Accuracy:  0.9753
Macro F1: 0.9757
```

Selected features:

```text
HTR
LWRB
PSGA
PSGB
RBLK
RM1
RM2
RM3
SGLK
STSG
STTB
TIME
VOID
WBK
WCFT
WECS
WFLB
WLPI
WRLA
WRLB
```

These results suggest that substantial feature reduction may be possible while preserving strong early-classification performance.

---

## Repository Structure

```text
.
├── README.md
├── requirements.txt
└── nuclear_accident_feature_reduction_clean.ipynb
```

---

## Installation

### Google Colab

Open the notebook and run all cells.

The notebook will install required dependencies and download the dataset automatically

### Local Environment

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook nuclear_accident_feature_reduction_clean.ipynb
```

---

## Limitations

This repository is intended as an exploratory research implementation

Known limitations include:

* Cluster formation depends on the selected correlation distance threshold, which may influence the resulting feature groups
* Pearson correlation captures linear relationships only
* Results are based on simulated accident data
* Additional validation across multiple random seeds would strengthen conclusions
* Nested cross-validation was not performed



## Attribution and Citation

### Original Dataset

If you use this repository, please cite the original NPPAD dataset source:

THU-INET
Nuclear Power Plant Accident Data (NPPAD)
https://github.com/thu-inet/NuclearPowerPlantAccidentData

### This Repository

This repository contains an independent feature-reduction and classification pipeline built on top of the publicly available NPPAD dataset.

All credit for dataset generation and publication belongs to the original NPPAD authors.
