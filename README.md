# PSMA-DVCHN

Official implementation of **PSMA-DVCHN**, a deep learning framework for small molecule–miRNA association prediction.

PSMA-DVCHN integrates conventional biological features with large language model (LLM)-derived semantic representations and jointly captures pairwise graph topology and higher-order structural relationships through graph convolutional and hypergraph convolutional networks.

---

## Overview

PSMA-DVCHN mainly consists of the following components:

- Conventional biological feature representation
- LLM-derived semantic feature representation
- Feature alignment and fusion
- Graph Convolutional Network (GCN)
- Hypergraph Convolutional Network
- Contrastive learning between graph views
- Learnable adaptive feature fusion
- Bilinear decoder for association prediction

The framework is designed to capture both pairwise association topology and higher-order structural information for small molecule–miRNA association prediction.

---

## Repository Structure

```text
PSMA-DVCHN/
├── code/
│   ├── main.py
│   ├── model.py
│   └── utils.py
│
├── data/
│   ├── dataset1/
│   │   ├── miRNA_feature.txt
│   │   ├── SM_feature.txt
│   │   ├── miRNA_llm_dataset1.txt
│   │   ├── SM_llm_dataset1.txt
│   │   └── miRNA_SM_adj.txt
│   │
│   └── dataset2/
│       ├── miRNA_feature.txt
│       ├── SM_feature.txt
│       ├── miRNA_llm_dataset2.txt
│       ├── SM_llm_dataset2.txt
│       └── miRNA_SM_adj.txt
│
└── README.md
```

---

## Datasets

Two dataset configurations are provided in this repository.

### Dataset 1

- 541 miRNAs
- 831 small molecules
- 664 known small molecule–miRNA associations

### Dataset 2

- 286 miRNAs
- 39 small molecules
- 664 known small molecule–miRNA associations

For each dataset, the association matrix is stored in:

```text
miRNA_SM_adj.txt
```

where:

- `1` represents a known small molecule–miRNA association.
- `0` represents an unconfirmed small molecule–miRNA pair.

The conventional biological features are stored in:

```text
miRNA_feature.txt
SM_feature.txt
```

The LLM-derived semantic representations are stored in:

```text
miRNA_llm_dataset1.txt
SM_llm_dataset1.txt
```

or:

```text
miRNA_llm_dataset2.txt
SM_llm_dataset2.txt
```

The two feature sources are separately processed and aligned before being concatenated to construct the final node representations.

---

## Model Architecture

PSMA-DVCHN adopts a dual-branch graph representation architecture.

### GCN Branch

The GCN branch captures pairwise topological relationships based on known small molecule–miRNA associations.

### Hypergraph Branch

The hypergraph branch models higher-order relationships among nodes. The hypergraph is constructed according to feature similarity using a K-nearest-neighbor strategy.

### Contrastive Learning

Contrastive learning is introduced to encourage representation consistency between the GCN view and the hypergraph view.

### Adaptive Fusion

The representations learned from the two branches are integrated using a learnable fusion gate:

```text
Z = α · Z_GCN + (1 - α) · Z_Hypergraph
```

where `α` is automatically optimized during training.

### Bilinear Decoder

A bilinear decoder is used to calculate association scores between small molecules and miRNAs.

---

## Environment

The implementation was tested with the following environment:

- Python: 3.8.10
- PyTorch: 1.11.0+cu113
- PyTorch Geometric: 2.1.0
- NumPy: 1.24.2
- Pandas: 2.0.3
- scikit-learn: 1.2.2
- CUDA: 11.3

A CUDA-enabled GPU is recommended for training.

The main Python dependencies can be installed using:

```bash
pip install numpy==1.24.2
pip install pandas==2.0.3
pip install scikit-learn==1.2.2
pip install torch-geometric==2.1.0
```

The experiments reported in this work were conducted using:

```text
PyTorch 1.11.0+cu113
CUDA 11.3
```

Please install the PyTorch build compatible with your CUDA environment before installing PyTorch Geometric.

---

## Installation

Clone this repository:

```bash
git clone https://github.com/mianbao621/PSMA-DVCHN.git
```

Enter the project directory:

```bash
cd PSMA-DVCHN
```

Run the following commands from the root directory of the repository.

---

## Usage

### Dataset 1

```bash
python code/main.py --dataset 1
```

### Dataset 2

```bash
python code/main.py --dataset 2
```

The default parameters in `main.py` correspond to the configuration used to obtain the main experimental results.

---

## Default Hyperparameters

| Parameter | Value |
|---|---:|
| Random seed | 2028 |
| Input dimension | 1024 |
| Hidden dimension | 128 |
| Output dimension | 128 |
| Learning rate | 0.001 |
| Weight decay | 5e-5 |
| Maximum epochs | 1000 |
| Warm-up epochs | 100 |
| Early stopping patience | 50 |
| Number of folds | 5 |
| Validation ratio | 0.1 |
| Contrastive loss weight λ | 1.0 |
| Temperature τ | 0.1 |
| Training positive:negative ratio | 1:1 |
| Hypergraph K | 10 |
| Dropout | 0.5 |

During the first 100 epochs, the model is optimized using the contrastive learning objective.

After the warm-up stage, the association prediction loss and contrastive learning loss are jointly optimized.

---

## Evaluation Protocol

PSMA-DVCHN is evaluated using five-fold cross-validation.

For each fold:

1. Known small molecule–miRNA associations are divided into training and test subsets.
2. A validation subset is further sampled from the training portion.
3. Positive and negative samples are evaluated at a balanced 1:1 ratio.
4. Training negative samples are dynamically sampled during model training.
5. Early stopping is performed according to validation AUC.
6. The model with the best validation AUC is restored for final testing.

The maximum number of training epochs is 1000, and the early-stopping patience is set to 50.

---

## Evaluation Metrics

The following evaluation metrics are reported:

- Area Under the ROC Curve (AUC)
- Area Under the Precision-Recall Curve / Average Precision (AUPR/AP)
- Accuracy (ACC)
- Sensitivity (SEN)
- Precision (PRE)
- Specificity (SPE)
- F1-score
- Matthews Correlation Coefficient (MCC)

---

## Main Results

### Dataset 1

| Metric | Mean ± Std |
|---|---:|
| AUC | 0.9936 ± 0.0026 |
| AUPR | 0.9941 ± 0.0024 |
| ACC | 0.9314 ± 0.0104 |
| SEN | 0.8644 ± 0.0182 |
| PRE | 0.9982 ± 0.0036 |
| SPE | 0.9985 ± 0.0030 |
| F1 | 0.9264 ± 0.0119 |
| MCC | 0.8708 ± 0.0192 |

### Dataset 2

| Metric | Mean ± Std |
|---|---:|
| AUC | 0.9778 ± 0.0032 |
| AUPR | 0.9700 ± 0.0082 |
| ACC | 0.9097 ± 0.0115 |
| SEN | 0.8630 ± 0.0368 |
| PRE | 0.9526 ± 0.0139 |
| SPE | 0.9563 ± 0.0146 |
| F1 | 0.9048 ± 0.0148 |
| MCC | 0.8239 ± 0.0188 |

---

## Notes

This repository provides the core implementation and datasets required to reproduce the main experiments of PSMA-DVCHN.

The current release focuses on the main model implementation. Additional experimental scripts are not included, including:

- Ablation studies
- Parameter sensitivity analysis
- Robustness experiments
- Positive-edge removal experiments
- Pseudo-positive perturbation experiments
- Entity-level cold-start experiments

These experiments are used for additional analyses and are not required to run the main PSMA-DVCHN model.

---

## Contact

For questions regarding the implementation or datasets, please open an issue in this repository.

Repository:

https://github.com/mianbao621/PSMA-DVCHN
