# **MR-ART: Hybrid 2D \+ 3D Gradient-Based MRI Quality Classification with Fusion**

## **Overview**

This project implements a **hybrid MRI quality classification framework** that combines:

* **2D slice-based gradient features**  
* **3D cuboid-based gradient features**  
* **MLP (Multi-Layer Perceptron) classifier**  
* **Decision-level fusion strategy**

The system is designed to classify structural brain MRI scans into:

* **Class 1** → High-quality scans (Score \= 1\)  
* **Class 2** → Lower-quality scans (Score \= 2 or 3\)

The framework is evaluated on:

* Internal dataset (`dataset_2_masked`)  
* External unseen-site dataset (ABIDE samples)  
* Additional external dataset (`Dataset_1_preprocessed`)

---

# **1\. Methodology**

## **1.1 2D Slice-Based Gradient Method**

### **Process**

* Extract 40 slices per orientation:  
  * Axial  
  * Sagittal  
  * Coronal  
* Perform percentile-based intensity normalization (1st–99th)  
* Compute gradient magnitude  
* Compute histogram-based difference metric (first 5 bins)  
* Form a **40 × 3 feature matrix per volume**  
* Train an **MLP classifier**  
* Apply **majority voting per volume**

### **MLP Architecture**

`MLPClassifier(`  
    `hidden_layer_sizes=(16, 8),`  
    `activation='relu',`  
    `solver='adam',`  
    `learning_rate='adaptive',`  
    `max_iter=500`  
`)`

Cross-validation is performed using 10-fold KFold.

---

## **1.2 3D Cuboid-Based Gradient Method**

### **Process**

* Divide volume into overlapping cuboids:  
  * Size: `(96, 128, 128)`  
  * Step size: 80% overlap  
* Normalize non-zero voxels  
* Compute 3D gradient magnitude  
* Compute histogram-based difference metric  
* Average across cuboids (27-region aggregation)  
* Compute ROC curve  
* Determine **optimal threshold using Youden’s Index**

This produces a confidence score per volume.

---

## **1.3 Fusion Strategy**

The final prediction combines:

* MLP prediction (2D)  
* 3D gradient threshold-based prediction

Fusion rule:

`If MLP predicts Class 2 OR gradient > threshold:`  
    `Predict Class 2`  
`Else:`  
    `Predict Class 1`

Confidence score is the average of:

* MLP probability  
* Sigmoid-transformed gradient distance

---

# **2\. Project Structure**

`mr_art(2d+3d_27avg)+abidetest.py`

Main sections in script:

1. 2D feature extraction  
2. 3D feature extraction  
3. MLP training  
4. Fusion evaluation  
5. ABIDE 50-sample testing  
6. ABIDE 100-sample testing  
7. Dataset\_1 (200 samples) testing

---

# **3\. Datasets Used**

## **3.1 Internal Dataset**

`/content/drive/MyDrive/dataset_2_masked/`

Includes:

* Masked T1-weighted NIfTI volumes  
* `scores.tsv` file with labels (1, 2, 3\)

## **3.2 External Datasets**

### **ABIDE Test Samples**

`/content/drive/MyDrive/ABIDE_testsamples/`

* class1/  
* class2/

---

# **4\. Requirements**

Install dependencies:

`pip install nibabel pandas numpy matplotlib seaborn scikit-learn scipy`

If running in Google Colab:

`!pip install nibabel`  
`from google.colab import drive`  
`drive.mount('/content/drive')`

---

# **5\. Evaluation Metrics**

The model reports:

* Accuracy  
* Confusion Matrix  
* Classification Report (Precision, Recall, F1-score)  
* ROC Curve (3D method)  
* Youden’s optimal threshold

---

# **6\. Testing Scenarios**

| Dataset | Samples | Type |
| ----- | ----- | ----- |
| Internal split | 200 volumes | Train/Test |
| ABIDE | 50 samples | Random unseen |
| ABIDE | 100 samples | Full set |
| Dataset\_1 | 200 samples | External |

---

# **7\. Key Contributions**

* Hybrid 2D \+ 3D gradient-based feature extraction  
* Histogram-based edge variation metric  
* Majority voting per volume  
* Youden-index optimized threshold  
* Confidence-based decision fusion  
* Robust external validation

---

# **8\. How to Run**

1. Mount Google Drive (if using Colab)  
2. Ensure dataset paths match your directory  
3. Run the script sequentially  
4. Review printed results and confusion matrices

---

# **9\. Classification Definition**

| Label | Meaning |
| ----- | ----- |
| Class 1 | Score \= 1 (High quality) |
| Class 2 | Score \= 2 or 3 (Lower quality) |

---

 **10\. Future Improvements**

* Replace MLP with CNN or 3D CNN  
* Add cross-site domain adaptation  
* Use SHAP for interpretability  
* Perform k-fold cross-site validation  
* Add automatic slice selection