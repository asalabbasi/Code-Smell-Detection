# Code-Smell-Detection
Multimodal code smell detection and severity classification for Java using a Wide &amp; Deep Gated Longformer that fuses raw source code (via Longformer's 4096-token attention) with 242 static analysis features from PMD. Trained and evaluated on the MLCQ dataset with soft labels and ordinal regression across four severity levels.
# Project Overview
Code smells—indicators of deeper design flaws in software—are notoriously difficult to detect due to subtle contextual patterns and highly imbalanced datasets. This project aims to accurately identify these smells (specifically the 'blob' smell type) by leveraging both static code features and deep contextual embeddings[cite: 1].
# Data Integration
The pipeline seamlessly maps raw source code to its corresponding metrics and labels using a precise merging strategy:

Static Features: Extracted from the PMDExtractedFeatures.csv dataset[cite: 1].

Expert Labels: Sourced from the MLCQCodeSmellSamples.csv dataset[cite: 1].

The Merge: The raw .java file extensions are stripped from the PMD ID column, converting it into an integer sample_id[cite: 1]. This sample_id acts as the primary key to join the PMD metrics perfectly with the MLCQ severity labels (None, Minor, Major, Critical)[cite: 1].
# Addressing Class Imbalance
Code smell datasets inherently feature a massive overrepresentation of "clean" code. To strictly raise the F1-score and prevent the model from defaulting to the majority class, we employ the following strategies:
1. Square-Root Class Weighting: Applied directly to the loss function to dynamically penalize majority class predictions. The optimized weights utilized in this pipeline are scaled to a baseline of None: 1.00, escalating to Critical: 13.19[cite: 1].
2. SMOTE (Synthetic Minority Over-sampling Technique): Applied exclusively to the training set during the tabular baselining phase to generate synthetic representations of the minority "smelly" classes.
3. Focal Loss Application: For advanced models, standard Cross-Entropy is swapped for Focal Loss, focusing the optimizer heavily on hard-to-classify "smelly" border cases.
# Feature Optimization
Not all PMD features carry equal predictive weight. To reduce noise, prevent overfitting, and ensure the model focuses strictly on structural complexities:
* Tree-Based Feature Importance: Utilizing Random Forest to establish a baseline of Gini importance for all PMD features.
* SHAP (SHapley Additive exPlanations): Applied to extract the marginal contribution of each tabular feature, allowing us to drop low-variance metrics that do not contribute to the DS1 boundary.
* Recursive Feature Elimination (RFE): Iteratively removing the weakest features until the optimal subset that maximizes the validation F1-score is identified.
# Model Architecture & Tuning
This project contrasts highly optimized tree-based algorithms against a multi-modal deep learning architecture.
1. Advanced Tree-Based Baselines
To establish a powerful tabular baseline, we utilize XGBoost and LightGBM. These algorithms excel at handling the dense, non-linear feature space of PMD metrics.
2. Wide & Deep Gated Longformer (SOTA)
Our primary architecture is a custom WideAndDeepGatedLongformer[cite: 1].

* The Deep Path: A Longformer model (configured for a maximum of 4096 tokens) generates embeddings directly from the raw Java source code[cite: 1].

* The Wide Path: Tabular PMD features are processed through a dedicated Multi-Layer Perceptron (MLP)[cite: 1].

* Fusion: Cross-attention mechanisms and a gating network dynamically weigh the importance of the text embeddings versus the static metrics before outputting the final ordinal classification[cite: 1].
# Evaluation Metrics
Model performance is evaluated against two primary operational scenarios:

* DS1 (Detection): A binary classification task separating cleanly written code from code containing any degree of smell[cite: 1]. Success here is measured strictly by the F1-Score and Matthews Correlation Coefficient (MCC)[cite: 1].

* DS2 (Prioritization): A classification task distinguishing between serious architectural flaws (Major/Critical) and less severe instances[cite: 1].

The pipeline dynamically tunes the classification threshold (e.g., shifting the probability threshold to 0.50 or 0.55) to maximize the MCC and F1 on the validation set[cite: 1].
