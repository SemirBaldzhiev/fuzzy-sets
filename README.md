# Brain MRI Segmentation and Tumor Detection System
**Author:** Semir Baldzhiev  
**Course:** Fuzzy Sets  
**University:** FMI at Sofia University  
**Date:** February 2026  

---

## 1. Introduction
Magnetic Resonance Imaging (MRI) is a cornerstone of modern neurosurgery and oncology, offering unprecedented detail of soft tissues. However, the manual segmentation of brain tumors is a labor-intensive task, prone to human error and subjectivity. Automating this process is critical for efficient treatment planning and longitudinal monitoring of patient outcomes.

This project addresses the challenges of automated segmentation, specifically the "fuzzy" nature of tumor boundaries. Unlike healthy organs with distinct borders, brain tumors often infiltrate surrounding tissues, creating a transition zone known as peritumoral edema. By utilizing **Fuzzy Logic**, specifically the Fuzzy C-Means (FCM) algorithm, we can model these uncertainties more accurately than traditional "hard" clustering methods.

---

## 2. Problem Formulation
The primary objective of this project is to develop a software system capable of autonomously isolating tumor masses from axial MRI slices. Mathematically, this is defined as partitioning the image pixels into distinct, semantically meaningful clusters based on their intensity profiles.

The main technical hurdle is the presence of the skull and scalp, which exhibit signal intensities nearly identical to the tumor in FLAIR (Fluid-Attenuated Inversion Recovery) sequences. The system must not only cluster pixels based on intensity but also differentiate between the skull and the tumor using morphological and spatial characteristics. Furthermore, the algorithm must effectively filter small noise fragments that often mimic tumor tissue in healthy slices without pathology. The ultimate goal is to create a reliable software tool that provides objective quantitative assessment of the lesion and facilitates the work of medical professionals.



---

## 3. Data Description
The system was validated using a high-quality dataset sourced from **The Cancer Imaging Archive (TCIA)**, specifically corresponding to 110 patients from **The Cancer Genome Atlas (TCGA)** lower-grade glioma (LGG) collection. This dataset is a benchmark in neuro-oncology research (Buda et al., 2019; Mazurowski et al., 2017).

* **Imaging Modality:** The dataset consists of axial MRI slices in the **FLAIR** sequence, optimized for detecting abnormalities.
* **Segmentation Masks:** Each image is accompanied by a manual FLAIR abnormality segmentation mask, providing a "gold standard" (Ground Truth) for performance evaluation.
* **Clinical Context:** The data is linked to the large-scale study of diffuse lower-grade gliomas published in the *New England Journal of Medicine* and includes additional genomic information for the patients.



---

## 4. Description of the Algorithm
The implemented pipeline follows a multi-stage approach to transform raw intensity data into a refined binary tumor mask.

### 4.1. Preprocessing
To improve the signal-to-noise ratio, the images undergo:
* **Gaussian Blurring:** To suppress high-frequency electronic noise.
* **CLAHE (Contrast Limited Adaptive Histogram Equalization):** To enhance local contrast, making the subtle differences between edema and tumor core more distinct.

### 4.2. Fuzzy C-Means Clustering (FCM)
The core of the segmentation is the FCM algorithm, which assigns a membership grade ($\mu$) to each pixel for every cluster. FCM minimizes the objective function:

$$J_m = \sum_{i=1}^{N} \sum_{j=1}^{C} \mu_{ij}^m \|x_i - c_j\|^2$$

We define **$C=4$ clusters**: Background, Healthy Tissue, Edema, and Tumor Core. This allows the model to handle the "partial volume effect" where a single pixel may contain both healthy and pathological tissue.



### 4.3. Defuzzification and Spatial Filtering
Post-clustering, the "fuzzy" membership matrix is converted into a "hard" label map using the **Maximum Membership** method via `np.argmax(u, axis=0)`. To eliminate the skull artifact, a balanced morphological filter is applied:
1.  **Connected Components:** The algorithm identifies all isolated "islands" in the tumor cluster.
2.  **Boundary Constraint:** Any component touching the edge of the frame (e.g., $x \approx 0$ or $x \approx 255$) is classified as skull/scalp and removed.
3.  **Area Thresholding:** Components smaller than 200 pixels are discarded to eliminate noise.

---

## 5. Results and Evaluation
The performance is quantified using the **Dice Similarity Coefficient (DSC)** and the **Fuzzy Partition Coefficient (FPC)**.

* **Mathematical Accuracy:** $DSC = \frac{2 |A \cap B|}{|A| + |B|}$.
* **Healthy Slice Handling:** By implementing area threshold and edge-constraint logic, the system achieved a **1.0 (perfect) score** on slices without tumors, effectively solving the "False Positive" problem.
* **Visual Validation:** The generated color maps provide a clear distinction between background (black), healthy tissue (blue), edema (green), and tumor core (red), matching anatomical distributions.



---

## 6. Software Implementation
The project is implemented in **Python 3.x** using a modular architecture.

### 6.1. Core Libraries
* `opencv-python`: Image manipulation and morphological operations.
* `scikit-fuzzy`: Implementation of the C-Means clustering algorithm.
* `numpy`: High-performance matrix operations and data reshaping.

### 6.2. Key Logic
Defuzzification is performed by selecting the cluster with the highest membership value for each pixel, followed by a `.reshape()` operation to restore the 2D spatial structure of the MRI slice. The filtering logic utilizes `cv2.connectedComponentsWithStats` to analyze the geometry of each candidate blob.



---

## 7. References
1.  Buda, M., Saha, A., & Mazurowski, M. A. (2019). "Association of genomic subtypes of lower-grade gliomas with shape features..." *Computers in Biology and Medicine*.
2.  Mazurowski, M. A., et al. (2017). "Radiogenomics of lower-grade glioma..." *Journal of Neuro-Oncology*.
3.  Bezdek, J. C. (1981). *Pattern Recognition with Fuzzy Objective Function Algorithms*.
4.  OpenCV Documentation: *Morphological Transformations*.