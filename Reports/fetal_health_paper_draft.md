# Fetal Health Classification Using Machine Learning

## Abstract

This study evaluates supervised learning models for fetal health classification using cardiotocographic (CTG) data and 21 original CTG predictors plus an engineered total-deceleration feature. Nine classifier families, including LightGBM, were compared, followed by class-imbalance, feature-selection, and hyperparameter-tuning experiments. On a stratified holdout set, tuned XGBoost achieved 95.31% accuracy and a macro F1-score of 0.9084. Its nested five-fold macro F1 on the training partition was 0.9274 (standard deviation 0.0058). The added feature did not change tuned XGBoost's holdout macro F1 compared with the prior 21-feature run; nested-CV macro F1 increased from 0.9240 to 0.9274.

## 1. Introduction

Fetal health monitoring is essential for identifying abnormalities during pregnancy and labor. Cardiotocography provides a non-invasive method for measuring fetal heart rhythm and related physiological indicators [1, 2]. However, interpreting CTG signals is often subjective and depends heavily on clinical expertise. Machine learning offers a data-driven alternative that can reduce variability and support clinical decision-making.

This paper investigates the performance of multiple machine learning classifiers for fetal health prediction. The study compares several widely used algorithms on the same CTG dataset and reports their predictive performance using standard classification metrics.

## 2. Methodology

The `Raw Data` sheet of `CTG.xls` contains 2,126 labeled observations. The target was `NSP` (classes 1, 2, and 3). The 21 original predictors were `LB`, `AC`, `FM`, `UC`, `DL`, `DS`, `DP`, `ASTV`, `MSTV`, `ALTV`, `MLTV`, `Width`, `Min`, `Max`, `Nmax`, `Nzeros`, `Mode`, `Mean`, `Median`, `Variance`, and `Tendency`. One engineered predictor, `TotalDecelerationBurden = DL + DS + DP`, was added, giving 22 model inputs. Metadata, derived classification columns, and all other workbook columns were excluded. The selected predictors had no missing values in the loaded data; model pipelines nevertheless use median imputation where applicable.

A stratified 80/20 train-test split was used to preserve the class proportions in the training and test partitions (random state 42), yielding 1,700 training and 426 test observations. Nine classifiers were evaluated: Logistic Regression, KNN, SVM, Decision Tree, Random Forest [3], Gradient Boosting [6], XGBoost [4], LightGBM [5], and MLP. Five-fold stratified cross-validation on the training partition ranked the candidates; randomized search with three-fold stratified inner CV tuned the two leading models. Their tuning procedures were assessed with nested five-fold stratified CV on the training partition. The test partition was not used for hyperparameter selection. Accuracy, macro precision, macro recall, macro F1, and class-specific recall were reported.

## 3. Results

Table 1 reports the nine-model holdout results using the fixed feature set. Class-2 recall is included because macro averages alone do not show the minority-class behavior.

\begin{table}[h]
\centering
\caption{Comparative performance of classifiers for fetal health classification}
\begin{tabular}{lccccc}
\hline
Model & Accuracy & Precision (macro) & Recall (macro) & F1-score (macro) & Recall (class 2) \\
\hline
XGBoost & 0.9460 & 0.9185 & 0.8863 & 0.9009 & 0.7627 \\
Gradient Boosting & 0.9437 & 0.9072 & 0.8807 & 0.8913 & 0.7458 \\
LightGBM & 0.9437 & 0.9092 & 0.8807 & 0.8930 & 0.7458 \\
Random Forest & 0.9272 & 0.8690 & 0.8969 & 0.8816 & 0.8305 \\
Decision Tree & 0.9178 & 0.8560 & 0.8526 & 0.8541 & 0.7458 \\
MLP & 0.9178 & 0.8559 & 0.8348 & 0.8446 & 0.7119 \\
SVM & 0.8685 & 0.7673 & 0.8431 & 0.7912 & 0.8814 \\
Logistic Regression & 0.8850 & 0.8025 & 0.7603 & 0.7767 & 0.6780 \\
KNN & 0.8826 & 0.7970 & 0.7260 & 0.7568 & 0.5254 \\
\hline
\end{tabular}
\end{table}

XGBoost and Gradient Boosting had similar baseline macro F1 scores. LightGBM achieved 0.9437 holdout accuracy and 0.8930 macro F1; its five-fold training CV macro F1 was 0.9190 (SD 0.0074), ranking it third among the nine models. XGBoost and Gradient Boosting were selected for tuning. Tuned XGBoost reached 0.9531 holdout accuracy and 0.9084 macro F1, with class-2 recall of 0.8136. Its nested five-fold macro F1 was 0.9274 (SD 0.0058). Tuned Gradient Boosting achieved 0.9484 accuracy and 0.9011 macro F1, with nested-CV macro F1 of 0.9155 (SD 0.0129).

For the exploratory feature-selection experiment, Random Forest feature importance was calculated using only the training partition. XGBoost was then evaluated with the top 10 and top 15 ranked features against its all-22-feature baseline. The top-10 model achieved 0.9390 accuracy and 0.8862 macro F1; the top-15 model achieved 0.9437 accuracy and 0.8979 macro F1, compared with 0.9460 accuracy and 0.9009 macro F1 using all 22 features. The reduced-feature results were not independently cross-validated, so they do not establish that feature reduction improves generalization.

The Random Forest imbalance experiment compared no balancing, `class_weight="balanced"`, and SMOTE applied to training data. Unweighted Random Forest had 0.8631 holdout macro F1 and 0.6949 class-2 recall. Class weighting yielded 0.8816 holdout macro F1 and 0.8305 class-2 recall; SMOTE yielded 0.8816 holdout macro F1 and 0.8136 class-2 recall. Their five-fold training CV macro F1 scores were 0.8985 (SD 0.0091), 0.9019 (SD 0.0122), and 0.9118 (SD 0.0142), respectively. Class weighting and SMOTE had the same class-3 holdout recall (0.9143).

The baseline Random Forest confusion matrix, with actual classes as rows and predicted classes as columns in order 1, 2, 3, was:

\[
\begin{bmatrix}
314 & 16 & 2 \\
8 & 49 & 2 \\
1 & 2 & 32
\end{bmatrix}
\]

Training-only Random Forest impurity importance ranked `ASTV` (0.1313), `ALTV` (0.1201), `Mean` (0.1038), `AC` (0.0779), and `Median` (0.0758) highest. These values describe this fitted model and should not be interpreted as causal effects.

## 4. Conclusion

With the 22 predictors, tuned XGBoost had the strongest observed holdout macro F1 and the highest nested-CV mean among the two tuned candidates. Adding `TotalDecelerationBurden` left tuned XGBoost holdout macro F1 unchanged at 0.9084 compared with the earlier 21-feature run, while nested-CV macro F1 rose from 0.9240 to 0.9274. LightGBM was a competitive baseline but did not exceed the two leading models. The top-15 feature experiment performed slightly below the all-feature baseline, while top-10 performed worse. Class weighting and SMOTE improved Random Forest class-2 recall relative to its unweighted variant, with differing class-2 recall between them. These results are limited to one dataset and one held-out split; they do not establish clinical readiness or external generalization. Further work should validate the selected pipeline on independent data and review the full per-class error costs before clinical interpretation.

## References

1. Campos, D., & Bernardes, J. (2000). Cardiotocography [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C51S4N.
2. Ayres-de-Campos, D., Bernardes, J., Garrido, A., Marques-de-Sá, J. P., & Pereira-Leite, L. (2000). SisPorto 2.0: A program for automated analysis of cardiotocograms. Journal of Maternal-Fetal Medicine, 9(5), 311–318. https://doi.org/10.1002/1520-6661(200009/10)9:5<311::AID-MFM12>3.0.CO;2-9.
3. Breiman, L. (2001). Random forests. Machine Learning, 45(1), 5–32. https://doi.org/10.1023/A:1010933404324.
4. Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining, 785–794. https://doi.org/10.1145/2939672.2939785.
5. Ke, G., Meng, Q., Finley, T., Wang, T., Chen, W., Ma, W., Ye, Q., & Liu, T.-Y. (2017). LightGBM: A highly efficient gradient boosting decision tree. Advances in Neural Information Processing Systems, 30.
6. Friedman, J. H. (2001). Greedy function approximation: A gradient boosting machine. The Annals of Statistics, 29(5), 1189–1232. https://doi.org/10.1214/aos/1013203451.
7. Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: Synthetic minority over-sampling technique. Journal of Artificial Intelligence Research, 16, 321–357. https://doi.org/10.1613/jair.953.
