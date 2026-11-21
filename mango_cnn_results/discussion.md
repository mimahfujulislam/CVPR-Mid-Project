## Discussion and conclusion

### 1. Overfitting in Round 1
The basic CNN has **8,483,397 parameters** (almost all in its first fully-connected layer) but only 2,100 training images with no variation. Its training accuracy reached **99.6%**, while validation accuracy stayed around 87.6% over the last 5 epochs (best 90.22% at epoch 16). Validation loss was lowest at epoch 7 (0.352) and then **rose to 0.546** by the last epoch, while training loss fell to 0.016. The final train-validation gap is **10.7 percentage points**. This is clear overfitting: the model memorized the training images instead of learning general disease patterns.

### 2. Effect of augmentation, BatchNorm and dropout (Round 2)
The improved CNN has **4,585,221 parameters**, about half as many as the basic model, because its 4th pooling step shrinks the flattened feature vector from 32,768 to 16,384 values. Its final train-validation gap is only **0.2 points** (compared with 10.7 in Round 1), and training and validation curves stay together from epoch 29 onward.

On the test set, Round 2 improves accuracy by **7.34 percentage points (88.22% → 95.56%)** and macro F1 from **0.8812 to 0.9553**. It needs more epochs (best at epoch 42 vs 16) and more training time (3.9 vs 0.6 min), because augmented images are harder to fit. The early validation instability is explained in Section 9 (BatchNorm statistics settling while the learning rate is high).

### 3. Test-set performance of the final model
| Metric | Value |
|---|---|
| Test accuracy | **95.56%** (430 / 450 correct) |
| Macro precision / recall / F1 | 0.9576 / 0.9556 / 0.9553 |
| ROC AUC (one-vs-rest) | 0.992 to 1.000 for every class |

- **Best class: Die_Back** (100% recall, 90/90). Its fully dried, brown, curled leaves are very distinctive.
- **Healthy** (precision 1.000, recall 0.967): no diseased leaf was ever called healthy, which matters in practice because no infection is missed that way.
- **Worst class: Gall_Mid** (86.7% recall, 78/90). 12 of the 20 test errors are Gall_Mid leaves.
- **Lowest precision: Anthracnose** (0.898): 10 leaves from other classes were predicted as Anthracnose.
- Most frequent confusions (true → predicted): **Gall_Mid → Die_Back** (6), **Gall_Mid → Anthracnose** (5), **Bacterial_Canker → Anthracnose** (3).

### 4. Why these errors happen (from the misclassified images)
- **Gall_Mid → Die_Back (6 images):** all six are fully dried, brown, curled leaves. Visually they are identical to Die_Back leaves, and the model predicts Die_Back with high confidence (64–94%). The dataset labels these as Gall Midge, perhaps a late stage of the infestation, but the image itself shows no gall marks. This is the largest single source of error, and it comes from overlap between the classes in the data, not from a model bug.
- **Gall_Mid / Bacterial_Canker ↔ Anthracnose (10 images):** these diseases all produce small dark or brown spots and lesions on green leaves. At 128×128 pixels the fine differences (raised galls vs sunken anthracnose lesions vs canker spots) are partly lost.
- **Healthy → disease (3 images):** dark-green leaves photographed at an angle or with shading. The model was unsure about these (37%, 46% and 81% confidence).
- Errors split into two kinds: **confident** mistakes (8 of 20 at ≥80% confidence, mostly the dried Gall_Mid leaves) and **uncertain** ones (4 of 20 at 50% or lower), where the model's top two classes were nearly tied.

### 5. Limitations
- All photos show single leaves on a plain white background, taken with smartphones in three regions of Bangladesh. Accuracy on leaves still on the tree, against cluttered backgrounds, is likely to be lower.
- Images are downscaled from 512×512 to 128×128, which removes detail from small lesions.
- The results come from one random 70/15/15 split with 450 test images, so each test error changes accuracy by 0.22 points. Repeating with several seeds or k-fold cross-validation would give error bars.
- Some labels overlap visually (dried Gall_Mid vs Die_Back leaves), which caps the achievable accuracy for any image-only model.

### 6. Future work
Higher input resolution (224×224) to keep lesion detail; transfer learning (ResNet, EfficientNet, or MobileNet for use on phones); Grad-CAM to check that the model looks at the lesions and not the background; reviewing the dried Gall_Mid images with an expert; k-fold cross-validation.

### 7. Conclusion
A CNN trained from scratch classifies the five MLD-BD leaf conditions with **95.56% test accuracy** and **0.955 macro F1**. Without regularization the network overfits heavily: a 10.7-point gap and 88.22% test accuracy. Data augmentation applied to the training split only, BatchNorm, dropout, weight decay and a cosine learning-rate schedule close that gap and add **7.34 points** of test accuracy, while using half as many parameters. The remaining errors come mostly from Gall_Mid leaves that look like Die_Back or Anthracnose.
