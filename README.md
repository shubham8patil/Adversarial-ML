# Adversarial Machine Learning

*Decision-Time (Evasion) Attacks and Data-Poisoning Attacks: Twenty Attack and Twenty Defense Strategies*

## 1. Introduction

Adversarial machine learning studies how learning systems can be manipulated by an adversary, and how such systems can be made robust to those manipulations. Attacks against machine learning models are broadly grouped into two categories: decision-time (evasion) attacks, in which an adversary perturbs an input at inference time to force a misclassification, and data-poisoning attacks, in which an adversary corrupts the training data so that the resulting model is degraded or contains a hidden vulnerability. This report presents an end-to-end empirical study covering 20 distinct attacks (15 evasion, 5 poisoning) and 20 corresponding defenses (10 against evasion, 10 against poisoning), implemented and evaluated across seven models spanning classical machine learning, clustering, and deep learning.

The study also satisfies two additional CIA rubrics. Rubric 1 (probability-based decision analysis) is addressed through confidence-distribution histograms, Bayesian prior/posterior analysis, prediction-entropy measurement, and ROC/AUC evaluation of the four classical classifiers. Rubric 2 (linear-model prediction) is addressed through a dedicated analysis of Logistic Regression: its learned coefficients, decision equation, and a 2-D PCA decision-boundary visualization.

### 1.1 Objectives

- Train four ML classifiers, two clustering techniques, and one deep-learning model (CNN) as attack/defense targets.
- Implement 15 evasion attacks and 5 data-poisoning attacks against these models.
- Implement 10 defenses against evasion and 10 defenses against poisoning, and quantify their effectiveness.
- Perform probability-based decision analysis (Rubric 1) and linear-model interpretation (Rubric 2).

## 2. Datasets

Two open-source, widely used benchmark datasets were used so that both classical/statistical models and a deep convolutional network could be evaluated on data appropriate to each.

| **Dataset** | **Domain** | **Train / Test Split** | **Used For** |
|---|---|---|---|
| MNIST (handwritten digits) | Image (28×28 grayscale) | 10,000 / 2,000 | CNN evasion, poisoning & defenses |
| Breast Cancer Wisconsin (Diagnostic) | Tabular, 30 numeric features | 455 / 114 | Classical ML & clustering attacks/defenses |

The Breast Cancer dataset has 212 malignant and 357 benign records (test set class balance was preserved via stratified split). MNIST was sub-sampled to 10,000 training and 2,000 test images to keep GPU training time manageable while retaining representative digit variety.

## 3. Models and Baseline Performance

Four classical classifiers and two clustering techniques were trained on Breast Cancer Wisconsin, and a 2-layer convolutional neural network (CNN) was trained on MNIST. Baseline (clean, unattacked) performance is summarized below and forms the reference point for every attack and defense result reported in this document.

| **Model** | **Category** | **Dataset** | **Clean Accuracy** |
|---|---|---|---|
| Logistic Regression | Linear classifier | Breast Cancer | 0.9825 |
| SVM (RBF kernel) | Classical ML | Breast Cancer | 0.9825 |
| Random Forest | Ensemble ML | Breast Cancer | 0.9561 |
| Gradient Boosting | Ensemble ML | Breast Cancer | 0.9561 |
| K-Means (k = 2) | Clustering | Breast Cancer | 0.9035 (test) |
| DBSCAN | Clustering | Breast Cancer | 0.7011 (train) |
| CNN (2-layer ConvNet) | Deep learning | MNIST | 0.9720 |

*ur classifiers. Right: ROC curves with AUC scores.*

## 4. Attacks (20 Total)

Fifteen decision-time (evasion) attacks and five data-poisoning attacks were implemented. Evasion attacks were run against the CNN (gradient-based: FGSM, PGD, BIM, DeepFool, C&W L2, JSMA; noise-based: Gaussian, Salt & Pepper, Uniform) and against each classical classifier / clustering method individually. Poisoning attacks corrupt training data before the model is fit.

| **Attack** | **Type** | **Target** | **Key Result** |
|---|---|---|---|
| 1. FGSM | Evasion | CNN | Acc 0.972 → 0.694 at ε=0.15 |
| 2. PGD | Evasion | CNN | Acc 0.972 → 0.462 at ε=0.15 |
| 3. BIM | Evasion | CNN | Acc → 0.496 at ε=0.15 |
| 4. DeepFool | Evasion | CNN | Acc → 0.025 (minimal perturbation) |
| 5. C&W L2 | Evasion | CNN | Acc 0.938 → 0.031 (mean L2 = 2.13) |
| 6. JSMA | Evasion | CNN | Targeted success 0/50 (0%) |
| 7. Gaussian Noise | Evasion | CNN + ML | CNN 0.972 → 0.787 at σ=0.5 |
| 8. Salt & Pepper | Evasion | CNN | Acc → 0.693 at 30% pixels |
| 9. LR Gradient | Evasion | Log. Regression | Acc 0.983 → 0.807 at ε=1.0 |
| 10. SVM Boundary | Evasion | SVM | Acc 0.980 → 0.120 |
| 11. RF Feature Perturb | Evasion | Random Forest | Acc 0.956 → 0.623 at ε=2.0 |
| 12. GB Adversarial | Evasion | Gradient Boosting | Acc unchanged: 0.956 → 0.956 |
| 13. K-Means Evasion | Evasion | K-Means | Acc 0.904 → 0.781 at α=2.0 |
| 14. DBSCAN Density | Evasion | DBSCAN | Noise ratio 0.351 → 1.000 |
| 15. Uniform Noise | Evasion | CNN | Acc stable: 0.972 → 0.957 at ε=0.5 |
| 16. Label Flipping | Poisoning | All 4 ML models | Acc drop up to 15.5 pts at 30% flip |
| 17. Backdoor / Trojan | Poisoning | CNN | 100% attack success rate |
| 18. Clean-Label Poison | Poisoning | LR, Random Forest | LR drop 1.75 pts; RF unaffected |
| 19. Feature Poison | Poisoning | All 4 ML models | RF drop 0.87 pts; others ≈ unaffected |
| 20. Gradient Poison | Poisoning | CNN | Acc 0.972 → 0.975 (negligible) |

The gradient-based attacks (FGSM, PGD, BIM, DeepFool, C&W) are the most damaging to the CNN: DeepFool and C&W reduce accuracy to near zero using only minimal, targeted perturbations, while PGD is consistently stronger than FGSM at the same perturbation budget ε because it iterates the gradient step (Figure 3). Among classical models, the SVM boundary attack and the linear-gradient attack on Logistic Regression are notably effective, whereas Gradient Boosting proved robust to the numerical-gradient evasion attack used here. Among poisoning attacks, the backdoor/trojan attack is the most severe: a 3×3-pixel trigger inserted into 10% of training images causes 100% of triggered test images to be misclassified as the target label, while leaving clean-input accuracy almost unaffected (0.9775).

![Figure 3](media/image1.png)

*Figure 3. FGSM adversarial examples on a single MNIST digit at increasing ε; the model's prediction flips from '7' to '3' once the perturbation becomes visible to the eye.*

## 5. Defenses (20 Total)

Ten defenses target decision-time evasion attacks (evaluated primarily against FGSM at ε = 0.15 on the CNN) and ten defenses target data-poisoning attacks (evaluated primarily against 10–20% label-flipping on the classical classifiers).

| **Defense** | **Against** | **Key Result (Accuracy)** |
|---|---|---|
| 1. FGSM Adv. Training | Evasion | Clean 0.983; under FGSM 0.901 (vs 0.694 undefended) |
| 2. PGD Adv. Training | Evasion | Clean 0.978; under PGD 0.833 (vs 0.462 undefended) |
| 3. Defensive Distillation | Evasion | Clean 0.966; under FGSM 0.673 |
| 4. Feature Squeezing | Evasion | Best at 1-bit: clean 0.972, under FGSM 0.931 |
| 5. Spatial Smoothing | Evasion | 3×3 kernel: clean 0.957, under FGSM 0.696 |
| 6. JPEG Compression | Evasion | Q=75: clean 0.973, under FGSM 0.748 |
| 7. Randomized Smoothing | Evasion | σ=0.1: clean 0.972, under FGSM 0.738 |
| 8. Ensemble Defense | Evasion | Clean 0.967, under FGSM 0.782 (3-model vote) |
| 9. Gradient Regularization | Evasion | Clean 0.984, under FGSM 0.692 |
| 10. NN Pruning | Evasion | 30% pruned: clean 0.975, under FGSM 0.701 |
| 11. RONI | Poisoning | Poisoned 0.956 → defended 0.974 (2 samples removed) |
| 12. Isolation Forest | Poisoning | 0.983 maintained (69 samples flagged) |
| 13. Spectral Signatures | Poisoning | Poisoned 0.956 → defended 0.974 |
| 14. KNN Sanitization | Poisoning | Poisoned 0.956 → defended 0.983 |
| 15. CV Filtering | Poisoning | Poisoned 0.956 → defended 0.965 |
| 16. Robust Statistics | Poisoning | Defended 0.965 vs clean baseline 0.983 |
| 17. Bagging Defense | Poisoning | 20-model bag: 0.956 (no gain over single model) |
| 18. Data Augmentation | Poisoning | Poisoned 0.956 → augmented 0.965 |
| 19. LOF Filtering | Poisoning | Poisoned 0.983 → defended 0.974 (69 removed) |
| 20. Ensemble Voting | Poisoning | Single 0.956 → 5-model ensemble 0.965 |

PGD adversarial training is the strongest evasion defense observed, nearly doubling robust accuracy under a PGD attack (0.462 → 0.833) while keeping clean accuracy high (0.978). Feature squeezing at 1-bit depth is the best lightweight (no-retraining) defense against FGSM. For poisoning, KNN-based sanitization and RONI most fully recover clean-level accuracy by directly identifying and removing corrupted or mislabeled samples, whereas simple bagging with no sample screening provides little benefit on its own. Figure 4 summarizes the accuracy trends across the key evasion attacks and the effect of the top five evasion defenses against FGSM.

![Figure 4](media/image2.png)

*Figure 4. Top-left: FGSM vs PGD accuracy vs ε. Top-right: noise-based attack comparison. Bottom-left: label-flipping impact per classifier. Bottom-right: defended vs undefended accuracy under FGSM (ε=0.15) for the five strongest defenses.*

## 8. Conclusion — Key Findings

- **The main aim is to study Adversarial Machine Learning**, where an attacker tries to fool an ML model and we try to protect the model from such attacks.
- **Two types of attacks are covered:** decision-time/evasion attacks, where the input is changed during prediction, and data-poisoning attacks, where harmful data is added or modified during training.
- **Four ML classifiers are used:** Logistic Regression, SVM, Random Forest, and Gradient Boosting. This allows us to compare how different ML models react to attacks.
- **Two clustering techniques are also used:** K-Means and DBSCAN. These help demonstrate how adversarial changes can affect clustering and grouping of data.
- **A CNN deep-learning model is used with the MNIST handwritten-digit dataset.** This allows us to demonstrate modern adversarial attacks such as FGSM, PGD, BIM, DeepFool, and C&W.
- **A total of 20 attacks are demonstrated:** 15 are decision-time/evasion attacks and 5 are data-poisoning attacks. The effect of each attack is measured using model performance such as accuracy.
- **A total of 20 defense strategies are implemented:** 10 defenses protect against evasion attacks and 10 protect against poisoning attacks. The purpose is to check whether the defenses can recover model performance.
- **The probability requirement is handled using model confidence, Bayesian probabilities, entropy, and ROC/AUC.** In simple words, we check how confident the model is before and after an attack.
- **The linear-model requirement is handled using Logistic Regression.** Its coefficients and decision boundary are analyzed to understand how features influence the prediction and how an attacker can move a sample across the decision boundary.
- **The final goal is to compare attack effectiveness and defense effectiveness.** For example, the report shows that PGD reduces CNN accuracy significantly, while PGD adversarial training improves the model's robustness. Similarly, KNN sanitization and RONI are effective against poisoning.
