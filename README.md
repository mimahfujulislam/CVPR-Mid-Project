# Mango Leaf Disease Classification (MLD-BD)

A convolutional neural network built and trained **from scratch** in PyTorch (no pretrained weights) to classify mango leaf diseases. The notebook compares a basic CNN with an improved, regularized CNN and includes a full result analysis.

**Notebook:** [`23_51620_2_CNN_MLD_BD.ipynb`](23_51620_2_CNN_MLD_BD.ipynb)

## Dataset

**MLD-BD: A Comprehensive Image Dataset for Mango Leaf Disease Detection in Bangladesh**
Mendeley Data, Version 2 (29 April 2025): https://data.mendeley.com/datasets/rjr2jvhdfy/2
DOI: [10.17632/rjr2jvhdfy.2](https://doi.org/10.17632/rjr2jvhdfy.2) · License: CC BY 4.0

- Smartphone photos of mango leaves from Rajshahi, Chapainawabganj and Mymensingh, Bangladesh
- 5 classes: **Anthracnose, Bacterial Canker, Die Back, Gall Midge, Healthy**
- 512×512 RGB JPG images

**Images used:** the **3,000 original images** (600 per class) from `Main image File`. The dataset's 6,000 pre-augmented images were **not** used: they are altered copies of the originals, so splitting them randomly would leak copies of test images into training.

| Split | Images | Per class |
|---|---|---|
| Train | 2,100 (70%) | 420 |
| Validation | 450 (15%) | 90 |
| Test | 450 (15%) | 90 |

The split is stratified, and augmentation is applied **only to the training images**.

## Method

All images are converted to RGB and resized to 128×128.

| | Round 1: `CNN_Basic` | Round 2: `CNN_Improved` (final model) |
|---|---|---|
| Conv blocks | 3 (32 → 64 → 128 filters) | 4 (32 → 64 → 128 → 256 filters) |
| BatchNorm | ✗ | ✓ after every conv layer |
| Dropout | ✗ | 0.3 and 0.5 in the classifier head |
| Data augmentation | ✗ | random crop/zoom, horizontal + vertical flip, rotation ±20°, colour jitter |
| Optimizer | Adam, lr 0.001 | Adam, lr 0.0005, weight decay (L2) 0.0001 |
| LR schedule | constant | cosine decay to 0.000005 |
| Epochs | 20 | 50 |
| Trainable parameters | 8,483,397 | **4,585,221** |

Loss function: cross-entropy. The weights from the epoch with the best validation accuracy are kept for testing.

## Results (test set, 450 images)

| Model | Test accuracy | Macro precision | Macro recall | Macro F1 | Train − val gap (last epoch) |
|---|---|---|---|---|---|
| Round 1: CNN_Basic | 88.22% | 0.8810 | 0.8822 | 0.8812 | +10.7 points (overfits) |
| **Round 2: CNN_Improved** | **95.56%** | **0.9576** | **0.9556** | **0.9553** | +0.2 points |

The final model reached its best validation accuracy (95.11%) at epoch 42, with validation loss 0.1469.

### Per-class results (final model)

| Class | Precision | Recall | F1-score |
|---|---|---|---|
| Anthracnose | 0.8980 | 0.9778 | 0.9362 |
| Bacterial Canker | 0.9775 | 0.9667 | 0.9721 |
| Die Back | 0.9375 | 1.0000 | 0.9677 |
| Gall Midge | 0.9750 | 0.8667 | 0.9176 |
| Healthy | 1.0000 | 0.9667 | 0.9831 |

### Key findings

- **The basic CNN overfits.** Its training accuracy reaches 99.6% while validation accuracy stays around 88%, and validation loss rises again after epoch 7.
- **Regularization works.** Augmentation, BatchNorm, dropout, weight decay and the cosine schedule raise test accuracy by **7.34 points**, almost remove the train-validation gap, and use **46% fewer parameters**.
- **Most remaining errors involve Gall Midge** (12 of the 20 test errors):
  - Some Gall Midge images show dried, brown, curled leaves that look like Die Back.
  - Gall Midge, Anthracnose and Bacterial Canker all produce small dark spots, which are hard to tell apart at 128×128 pixels.

## What the notebook contains

1. Dataset loading from Google Drive and exploration (class counts, sample images)
2. Stratified train / validation / test split
3. Round 1: basic CNN, showing overfitting
4. Visualization of learned filters and feature maps
5. Round 2: improved CNN with augmentation, BatchNorm, dropout and an LR schedule
6. Result analysis:
   - loss and accuracy curves
   - comparison of the two rounds
   - confusion matrix and classification report
   - per-class accuracy
   - ROC curves (one-vs-rest)
   - sample predictions and misclassified images
  

## How to run

1. Download `Mango_Dataset.zip` from the [dataset page](https://data.mendeley.com/datasets/rjr2jvhdfy/2) and upload it to the top level of your Google Drive (*My Drive*).
2. Open the notebook in [Google Colab](https://colab.research.google.com/) and choose **Runtime → Change runtime type → T4 GPU**.
3. Click **Runtime → Run all** and allow Google Drive access when asked.

The notebook finds and unzips the dataset itself, ignoring the hidden `__MACOSX` folder. A full run takes about 8 minutes on a T4 GPU.

**Requirements** (all preinstalled on Colab): Python 3, PyTorch, torchvision, scikit-learn, NumPy, pandas, Matplotlib, seaborn, Pillow and torchsummary (installed by the notebook if missing).

> GPU training is not fully deterministic, so a new run gives slightly different numbers. The results above are from the saved run in this notebook.

## Citation

> Islam, M., Kowser, K. A., Ahad, M. T., Mustofa, S., Emon, Y. R., & Miah, M. S. (2025). *MLD-BD: A Comprehensive Image Dataset for Mango Leaf Disease Detection in Bangladesh* (Version 2) [Data set]. Mendeley Data. https://doi.org/10.17632/rjr2jvhdfy.2
