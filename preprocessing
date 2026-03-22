# Roadmap Checklists (Image Collection → Preprocessing → Pilot Labels → YOLO Baseline Comparison)

- Collect open street-level images (any area)
- Preprocess + organize into a clean dataset
- Label a small pilot set
- Train/compare 2–3 YOLO baselines and pick the bes

## Person 1 — Image Collection (Open imagery → raw images + manifest)

- [ ] Pick 2 small areas with good street-level imagery coverage.
- [ ] Save boundaries (bbox/polygon) + target counts (500–2,000 images/area).
- [ ] Pull image list per area (ID, lat/lon, timestamp, heading if available, source URL).
- [ ] Download images with stable filenames: `<source>_<image_id>.jpg`.
- [ ] Apply sampling limits (reduce redundancy): ~1 image per 10–20m when possible.
- [ ] Bias collection near intersections/crossings (needed for curb/no ramp).

## Person 2 — Preprocessing + Dataset Setup (clean → organize → split)

- [ ] Create canonical dataset structure: `dataset_v0/images/all/`, `dataset_v0/metadata/`, `dataset_v0/splits/`.
- [ ] Decide storage: cleaned-only vs cleaned+resized (prefer cleaned+resized if training speed matters).
- [ ] Preprocess all raw images:
  - [ ] drop corrupt/undecodable
  - [ ] ensure RGB
  - [ ] fix EXIF rotation
  - [ ] standardize format (JPEG or standardized copies)
  - [ ] remove exact duplicates (hash)
  - [ ] optional: near-duplicates (perceptual hash), blur/too-dark filters
- [ ] Merge manifests into `dataset_v0/metadata/images.csv|parquet`.
- [ ] Add derived fields: `preprocessed_path`, `quality_flags`, `geocell_id` (from lat/lon).
- [ ] Create splits by geocell_id (not random):
  - [ ] `train.txt`, `val.txt`, `test.txt`
- [ ] Select pilot subset (200–400 images) emphasizing:
  - [ ] intersections/crossings
  - [ ] older/rough sidewalks
  - [ ] lighting variety
- [ ] Produce `dataset_v0/metadata/pilot_subset.csv` + prepared folder for labeling.

## Person 3 — Annotation (label rules → pilot labels + QA)

- [ ] Write `docs/labeling_guide_v0.1.md` with rules for:
  - [ ] **stairs** (tight box; rule for single-step)
  - [ ] **curb/no ramp** (crossings only; how to handle occlusions; ramp-present = no label)
  - [ ] **broken pavement** (clear damage only; minimum size; ignore shadows/markings)
  - [ ] “do not label” examples + consistency checklist
- [ ] Set up tool project (CVAT/Label Studio/etc.) with 3 classes.
- [ ] Import pilot subset.
- [ ] Label **200–400 images**.
- [ ] Track counts per class; if curb/no ramp is too rare, request more intersection images.
- [ ] Fix systematic issues.
- [ ] Export YOLO labels.
- [ ] Validate:
  - [ ] image↔label file matching
  - [ ] class IDs match `class_map.json`
  - [ ] bboxes valid and normalized

## Person 4 — YOLO Baselines (protocol → train → compare → recommend)

- [ ] Create `experiments/exp_protocol_v0.md` fixing:
  - [ ] image size (e.g., 640 or 768)
  - [ ] epochs (50–100 or early stop)
  - [ ] augmentations
  - [ ] batch size / hardware info
- [ ] Define metrics: mAP@0.5, per-class precision/recall, inference speed, model size/memory.
- [ ] Choose **2–3 YOLO candidates**:
  - [ ] small/fast
  - [ ] medium/balanced
  - [ ] optional large/accurate (if hardware allows)
- [ ] Train each candidate on the same pilot split.
- [ ] Log: training time, final metrics, signs of overfit/instability.
- [ ] Error analysis: collect common failures:
  - [ ] broken pavement FP (shadows/textures)
  - [ ] curb/no ramp FN (small/occluded)
  - [ ] stairs FN (distance/angle)
  - [ ] confusions (curb edges vs cracks)
- [ ] Produce `experiments/results_table_v0.csv` + `experiments/error_analysis_v0.md`.
- [ ] Recommend one baseline (“Use YOLO ___ for v1”), with notes on accuracy vs speed.

Deliverables:
- [ ] A chosen baseline YOLO model + saved weights.
- [ ] A clear comparison table proving why it was chosen.
- [ ] A short list of next data actions to improve weakest class.
