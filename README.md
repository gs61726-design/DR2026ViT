# APA-SegFormer: Anatomically Guided Transformer for Diabetic Retinopathy Segmentation and Grading

This repository provides the reproducibility materials for the paper:

**"Transformer-Based Diabetic Retinopathy Segmentation and Grading: Anatomical Prior Attention for False-Positive Suppression"**

Shokhan M. Al-Barzinji, Hamsa M. Ahmed, M. M. Abdulrazzaq

*International Journal of Intelligent Engineering and Systems (IIJIES)*

---

## Repository Contents

```
APA-SegFormer-DR/
├── README.md
├── requirements.txt
├── configs/
│   └── train_config.yaml          # All hyperparameters, random seeds
├── splits/
│   ├── train_ids.txt              # 43 training image IDs
│   ├── val_ids.txt                # 11 internal validation image IDs
│   ├── test_ids.txt               # 27 independent test image IDs
│   └── fivefold/
│       ├── fold1_val_ids.txt      # Fold 1 validation IDs
│       ├── fold2_val_ids.txt      # Fold 2 validation IDs
│       ├── fold3_val_ids.txt      # Fold 3 validation IDs
│       ├── fold4_val_ids.txt      # Fold 4 validation IDs
│       └── fold5_val_ids.txt      # Fold 5 validation IDs
├── evaluation/
│   ├── eval_segmentation.py       # mDice, mIoU, per-class Dice, HD95, AUPR
│   ├── eval_classification.py     # Accuracy, QWK, Macro AUC, Sensitivity, Specificity
│   ├── eval_fpr.py                # OD-adjacent and vessel-adjacent FPR
│   └── wilcoxon_tests.py          # Statistical significance tests
├── preprocessing/
│   └── dtcwt_preprocessing.py     # Wavelet-guided preprocessing pipeline
└── priors/
    └── generate_priors.py         # Anatomical prior map generation
```

## Dataset

This study uses the **Refined IDRiD** dataset. The images are not included in this
repository due to third-party licensing. Please obtain the dataset from:

> T. Chankhachon, et al., "Refined IDRiD: An enhanced dataset for diabetic retinopathy
> segmentation with expert-validated annotations and comprehensive anatomical context",
> Data, Vol.11, No.2, Article 30, 2026. https://doi.org/10.3390/data11020030

## Frozen Anatomical Prior Extractors

The anatomical prior maps are generated using frozen pretrained models:

- **Optic Disc Segmenter**: SegFormer-based disc/cup segmenter trained on [REFUGE](https://refuge.grand-challenge.org/) (Dice = 0.95 on REFUGE validation set)
- **Vessel Segmenter**: [FR-UNet](https://github.com/lseventeen/FR-UNet) trained on [DRIVE](https://drive.grand-challenge.org/) (AUC = 0.97 on DRIVE test set)
- **Fovea**: Geometric estimate from disc centroid (rule-based)
- **Retinal Boundary**: Otsu thresholding + morphological operations (rule-based)

## Hardware and Software Environment

- **GPU**: NVIDIA A100 80 GB
- **Framework**: PyTorch 2.1, HuggingFace Transformers 4.36
- **Python**: 3.10+
- **Key packages**: See `requirements.txt`

## Reproducing Results

1. Obtain the Refined IDRiD dataset and place images under `data/refined_idrid/`
2. Generate anatomical priors:
   ```bash
   python priors/generate_priors.py --data_dir data/refined_idrid/ --output_dir data/priors/
   ```
3. Run preprocessing:
   ```bash
   python preprocessing/dtcwt_preprocessing.py --data_dir data/refined_idrid/ --output_dir data/preprocessed/
   ```
4. Evaluate segmentation:
   ```bash
   python evaluation/eval_segmentation.py --pred_dir results/predictions/ --gt_dir data/refined_idrid/masks/ --split_file splits/test_ids.txt
   ```
5. Evaluate classification:
   ```bash
   python evaluation/eval_classification.py --pred_file results/grading_preds.csv --gt_file data/refined_idrid/grades.csv --split_file splits/test_ids.txt
   ```
6. Evaluate false-positive rates:
   ```bash
   python evaluation/eval_fpr.py --pred_dir results/predictions/ --gt_dir data/refined_idrid/masks/ --prior_dir data/priors/ --split_file splits/test_ids.txt
   ```
7. Run statistical tests:
   ```bash
   python evaluation/wilcoxon_tests.py --results_dir results/
   ```

## Training Configuration

All hyperparameters are specified in `configs/train_config.yaml`. Key settings:
- Input resolution: 512×512
- Backbone: MiT-B3 (ImageNet pretrained)
- Optimizer: AdamW (lr=6e-5, weight_decay=0.01)
- Batch size: 4 (gradient accumulation: 4)
- Early stopping patience: 30 epochs (based on validation mDice)

## Note

The full training pipeline (APA module, AW-FTL loss, multi-task decoder) will be
released upon paper acceptance.

## Citation

```bibtex
@article{albarzinji2026apa,
  title={Transformer-Based Diabetic Retinopathy Segmentation and Grading: Anatomical Prior Attention for False-Positive Suppression},
  author={Al-Barzinji, Shokhan M. and Ahmed, Hamsa M. and Abdulrazzaq, M. M.},
  journal={International Journal of Intelligent Engineering and Systems},
  year={2026}
}
```

## License

This project is licensed under the MIT License.
