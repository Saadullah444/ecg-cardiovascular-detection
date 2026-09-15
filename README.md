# A Novel Transformer-Based Approach for Cardiovascular Disease Detection

This Code is for the paper published in *Frontiers in Digital Health* (2025).

> Noor N, Bilal M, Abbasi SF, Pournik O, Arvanitis TN. **A novel transformer-based approach for cardiovascular disease detection.** *Front. Digit. Health* 7:1548448. doi: [10.3389/fdgth.2025.1548448](https://doi.org/10.3389/fdgth.2025.1548448)

---

## Overview

This repository contains the full experimental pipeline for classifying four cardiovascular conditions from ECG derived features:

| Label | Condition |
|-------|-----------|
| `ARR` | Arrhythmia |
| `AFF` | Atrial fibrillation |
| `CHF` | Congestive heart failure |
| `NSR` | Normal sinus rhythm |

The method works in three stages:

1. **Feature selection.** A random forest is trained one vs. rest on 54 morphological, fiducial, statistical and heart rate variability (HRV) features. Features whose maximum class wise importance exceeds 0.02 are retained, which cuts the feature set down to the most prominent 13.
2. **Tokenisation.** Each retained feature vector is serialised into a text string of the form `QRtoQSdur: 0.001, RStoQSdur: 0.001, RRmean: 29...` and tokenised with the BERT WordPiece tokeniser (128 token sequences).
3. **Classification.** A transformer encoder with a sequence classification head is fine tuned on these tokenised feature strings for four way prediction.

Reported test accuracy is **99.79%**, with an AUC of 1.00 for three of the four classes.

## Repository structure

```
.
├── MainCodeforpaper.ipynb     # Full pipeline: EDA, imputation, feature selection, training, evaluation
├── ECGCvdata.csv              # Input dataset (see "Dataset" below, not included in this repo)
├── requirements.txt           # Python dependencies
└── README.md
```

Running the notebook end to end creates a `./my_ecg_filtered_classifier/` directory holding the fine tuned model weights and tokeniser configuration.

## Dataset

The study uses 1,200 ECG recordings (300 per class) sampled at 250 Hz, drawn from the MIT-BIH PhysioNet databases. The pre extracted feature table is publicly available on Kaggle:

**https://www.kaggle.com/datasets/akki2703/ecg-of-cardiac-ailments-dataset/data**

Download `ECGCvdata.csv` and place it in the repository root before running the notebook. The dataset is not redistributed here.

Feature groups in the CSV:

| Type | Count | Examples |
|------|-------|----------|
| Morphological | 11 | P duration, QRS duration, QRS area, PQR and QRS angles |
| Fiducial | 29 | Inter wave distances, PQ / QR / RS / ST slopes and intervals |
| Statistical | 4 | RR mean, PP mean, QR/QS ratio, RS/QS ratio |
| HRV | 9 | SDRR, NN50, pNN50, SDSD, RMSSD, RRTot, NNTot |
| Heart rate | 1 | Beats per minute |

Raw signals were band pass filtered with a fourth order Butterworth filter (0.5 to 150 Hz), and characteristic waves were located using a four level symlet maximal overlap discrete wavelet packet transform (MODWPT). That preprocessing produced the published CSV and is described in Section 3 of the paper.

## Requirements

Python 3.9 or newer.

```bash
pip install -r requirements.txt
```

`requirements.txt`:

```
pandas
numpy
matplotlib
seaborn
scikit-learn
torch
transformers<4.40
```

> **Note on `transformers`:** the notebook imports `AdamW` from `transformers`, which was removed in version 4.40. Either pin an earlier release as above, or swap the import for `from torch.optim import AdamW`, which is a drop in replacement. A GPU is optional. The notebook falls back to CPU automatically. 

## Running the pipeline

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install -r requirements.txt
jupyter notebook MainCodeforpaper.ipynb
```

Execute the cells in order. The stages are:

1. **Load and inspect.** Reads `ECGCvdata.csv`, prints summary statistics, plots a correlation heatmap over the 54 features and per class box plots.
2. **Impute missing values.** Nine columns (`QRtoQSdur`, `RStoQSdur`, `PonPQang`, `PQRang`, `QRSang`, `RSTang`, `STToffang`, `QRslope`, `RSslope`) contain gaps. These are filled with the class wise median.
3. **Rank features.** A `RandomForestClassifier` (100 trees) gives overall importances, then a `OneVsRestClassifier` wrapper gives per class importances. PCA is used here only for a 2D visual check of class separability.
4. **Select features.** Any feature whose maximum importance across the four classes exceeds 0.02 is kept. Values are rounded to three decimal places.
5. **Serialise to text.** Each row becomes a `name: value` string, labels are integer encoded, and the data is split 80/20 into train and test.
6. **Tokenise and fine tune.** `bert-base-uncased` with a four label classification head, AdamW optimiser, 5 epochs, batch size 2, maximum sequence length 128. Weights are written to `./my_ecg_filtered_classifier/`.
7. **Evaluate.** Reloads the saved model and produces accuracy, a full classification report, a confusion matrix, per class ROC curves and per class precision recall curves.

## Results

Per class performance on the held out test set:

| Metric | AFF | ARR | CHF | NSR |
|--------|-----|-----|-----|-----|
| Accuracy | 0.9958 | 1.0000 | 0.9958 | 1.0000 |
| Sensitivity | 1.0000 | 1.0000 | 0.9833 | 1.0000 |
| Specificity | 0.9944 | 1.0000 | 1.0000 | 1.0000 |
| Precision | 0.9836 | 1.0000 | 1.0000 | 1.0000 |
| F1 score | 0.9920 | 1.0000 | 0.9916 | 1.0000 |

**Overall test accuracy: 99.79%**

## Citation

If you use this code, please cite the paper:

```bibtex
@article{noor2025transformer,
  title   = {A novel transformer-based approach for cardiovascular disease detection},
  author  = {Noor, Nimra and Bilal, Muhammad and Abbasi, Saadullah Farooq and Pournik, Omid and Arvanitis, Theodoros N.},
  journal = {Frontiers in Digital Health},
  volume  = {7},
  pages   = {1548448},
  year    = {2025},
  doi     = {10.3389/fdgth.2025.1548448}
}
```

## Funding

This work was partially funded by:

- **MedSecurance** (Advanced Security-for-safety Assurance for Medical Device IoT), Grant Agreement 101095448
- **INSAFEDARE** (Innovative Applications of Assessment and Assurance of Data and Synthetic Data for Regulatory Decision Support), Grant Agreement 101095661

The authors acknowledge the University of Birmingham's BlueBEAR HPC service for the computing resources used in this study.

## Licence

The paper is distributed under the [Creative Commons Attribution License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). 

## Contact

**Saadullah Farooq Abbasi**
Department of Electronic, Electrical and Systems Engineering, University of Birmingham
s.f.abbasi@bham.ac.uk
