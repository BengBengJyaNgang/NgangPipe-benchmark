# NgangPipe

**NgangPipe** is the first large-scale public object-detection dataset with bounding-box annotations for CCTV imagery of urban drainage pipelines. It is released together with a benchmark of 21 YOLO-family detectors trained under a unified per-class protocol (378 training runs in total), providing a common foundation for algorithm research and reproducible comparison in intelligent sewer inspection.

- 📄 Paper: *NgangPipe: A Dataset and Evaluation Benchmark for CCTV Defect Detection in Urban Drainage Pipelines* (WACV 2027 UrbanState Workshop, Datasets Track, under review)
- 🧭 Dataset home (benchmark code + weights): https://github.com/BengBengJyaNgang/NgangPipe-benchmark
- 🤗 Dataset mirror: https://huggingface.co/datasets/bengbengJyaNgang/NgangPipe-benchmark

## Highlights

| | |
|---|---|
| Defect images (unique) | **25,267** |
| Bounding-box annotations | **34,370** |
| Defect classes | **18** (organized by pipe material × visual morphology) |
| Pipe materials | HDPE corrugated pipe & concrete pipe |
| Inspection mileage | 120 km of CCTV inspection video |
| Benchmark models | 21 YOLO variants (YOLOv5–YOLO26, six generations) |
| Training runs | 378 (per-class single-class protocol) |
| Overall mean mAP50 | **0.8304** |

Eight classes are near saturation (best per-class mAP50 ≥ 0.92), while filamentous tree roots, misaligned joints in concrete pipe, leakage, and related classes still show clear headroom — with **missed detection** as the dominant failure mode. Per-class results, failure analysis, confusion matrices, and data-scale ablations are reported in the paper.

## Taxonomy

The 18 classes are organized along two dimensions — pipe material and visual morphology — and keep an engineering-semantics mapping to CJJ 181-2012 (see the paper for the full mapping table). Group legend: **B** = HDPE corrugated pipe–specific, **T** = common to both pipe materials, **H** = concrete pipe–specific. DSW = Defect Severity Weight derived from CJJ 181-2012 grades.

| Code | Class name | Group | Images | Boxes | DSW |
|---|---|---|---:|---:|---:|
| 0BXX | Heart-shaped inward fold | B | 919 | 914 | 0.80 |
| 1BGK | Joint-ring anomaly | B | 913 | 1,491 | 0.40 |
| 2BTZ | Strip-shaped anomaly | B | 5,170 | 7,464 | 0.50 |
| 3BJD | Joint separation | B | 1,579 | 2,439 | 0.90 |
| 4BPD | Hole in HDPE corrugated pipe | B | 969 | 966 | 0.90 |
| 5BTL | Collapse of HDPE corrugated pipe | B | 972 | 982 | 1.00 |
| 6TZA | Obstacle | T | 2,360 | 2,434 | 0.20 |
| 7TCJ | Sediment deposit | T | 2,587 | 2,460 | 0.20 |
| 8TSL | Leakage | T | 706 | 785 | 0.50 |
| 9TAJ | Illicit branch connection | T | 1,636 | 1,667 | 0.20 |
| 10TQF | Water pooling | T | 894 | 847 | 0.10 |
| 11BLZ | Granular anomaly | B | 619 | 660 | 0.30 |
| 12HCK | Misaligned joint in concrete pipe | H | 683 | 1,321 | 0.50 |
| 13HLF | Crack in concrete pipe | H | 3,932 | 5,345 | 0.60 |
| 14HPD | Hole in concrete pipe | H | 663 | 646 | 0.90 |
| 15HTL | Joint-filler detachment | H | 534 | 575 | 0.10 |
| 16TSS | Filamentous tree roots | T | 1,433 | 2,429 | 0.40 |
| 17TTS | Mass tree roots | T | 594 | 945 | 0.50 |

&gt; Per-class counts are computed on decomposed single-defect training images; frames containing defects of multiple classes are counted once in each of their classes. About 7% defect-free background frames are additionally mixed into the training split at release to suppress false detections and are **not** counted in the table above.

## Dataset structure

The release package takes `NGANGPipe-dataset` as the root directory and is organized by defect class. Each class subset is released in YOLO format with an 8:1:1 train/val/test split. Ten classes also retain their raw (`_original`) data.

```text
NGANGPipe-dataset/
├── {class code}/
│   ├── {class code}_original/      # raw data, stored flat (10 classes only)
│   └── {class code}_Processed/     # release & training data
│       ├── images/{train, val, test}/
│       └── labels/{train, val, test}/   # YOLO TXT, single-class (class 0)
├── 18HNM/                          # background-frame banks (defect-free)
├── 19BNM/
└── weights/                        # benchmark model weights
```

- Annotations follow the standard YOLO format (`class x_center y_center width height`, normalized). Labels are **single-class within each subset** (every box is class `0`); the subset directory name is the class identity. The 18-class mapping is given in `classes.yaml`.
- Data-scale ablation subsets (100% / 75% / 50% / 25% / 12% of the training images) are provided for the reference model YOLO11-m.
- Multi-defect frames are decomposed into single-defect training images with selective Gaussian blur on non-target regions, so every training image carries exactly one bounding box and one unambiguous label.

## Quickstart

Environment used for the benchmark (any compatible PyTorch/Ultralytics stack works):

```bash
pip install ultralytics==8.4.138   # Python 3.9, PyTorch 1.13.1, CUDA 11.7
```

Train one class subset with its release split (example: `13HLF`):

```bash
yolo detect train \
  data=13HLF/13HLF_Processed/13HLF.yaml \
  model=yolo11m.pt \
  epochs=500 patience=100 batch=8 lr0=0.01
```

The benchmark protocol used in the paper: every model is initialized from COCO-pretrained weights and trained **per class** (single-class detection head) for 500 epochs with early-stopping patience 100, SGD optimizer (initial lr 0.01, framework-default momentum/weight decay), batch size 8, default data augmentation, and identical hyperparameters across all 378 runs. Validation metrics are computed on the validation split; the test split is reserved and not used for training or tuning.

## Benchmark protocol & metrics

- **Per-class training protocol.** Each of the 18 class subsets is trained independently, avoiding interference between visually similar classes on a shared classification head.
- **Metrics.** Per-class AP50 at IoU 0.5, and the 18-class macro-averaged mAP50. The dataset also supports mAP50:95, precision, recall, and F1.
- **Reference results.** Merged-training vs. per-class protocol comparison (YOLO11-m baseline) and per-class best models are listed in the paper (Tables 5–6); benchmark weights are in `weights/`.

## License and attribution

- The self-collected portion of the dataset (imagery and annotations) is released under the **CC BY-NC 4.0** license: free use, modification, and redistribution with attribution, **non-commercial** use only.
- The benchmark code is released under the **MIT** license.
- External concrete-pipe defect imagery is packaged separately under the source party's license and is **not** re-licensed under CC BY-NC 4.0.
- All images are embedded with invisible watermarks to track unauthorized distribution. Please do not attempt to reverse-engineer the real pipe locations corresponding to anonymized codes.
- The dataset is provided **"as is"**, without warranty of fitness for a particular purpose.

**Please cite the paper when publishing work based on this dataset.**

```bibtex
@inproceedings{ngangpipe2027,
  title   = {NgangPipe: A Dataset and Evaluation Benchmark for CCTV Defect Detection in Urban Drainage Pipelines},
  author  = {Anonymous},
  booktitle = {WACV 2027 UrbanState Workshop, Datasets Track},
  note    = {Under review},
  year    = {2027}
}
```

## Changelog

See Appendix C.1 of the paper for the dataset version change log.

## Issues

For dataset questions, mislabeled samples, or benchmark reproduction issues, please open an issue in this repository.
