# Terminology Glossary 

## Core dataset terms

### **Area / Area ID (`<area_id>`)**
A chosen geographic region where you collect images from (e.g., a neighborhood, a campus, a few streets).  
`<area_id>` is just a short unique name for that area (example: `area_01_downtown`).

### **Boundary (bbox/polygon)**
How you describe the area on a map:
- **bbox (bounding box):** a rectangle defined by min/max latitude and longitude.
- **polygon:** a more precise shape (multiple points) that follows the real boundary.

### **Open street-level imagery**
Publicly available street-view-like images from sources such as Mapillary or other open datasets.

### **Manifest (`manifest.csv`, `raw_manifest_<area_id>.csv`)**
A manifest is a table (usually CSV) with **one row per image** that acts as the dataset’s inventory.
It contains both:
- Where the image is on disk (`local_path`)
- Metadata about the image (GPS, time, source ID, etc.)

- Prevents “folder guessing” (pipelines read the manifest, not the directory listing).
- Enables reproducibility, debugging, and tracking licensing/attribution.

**Typical columns**
- `image_id`, `source`, `area_id`
- `local_path`
- `lat`, `lon`
- `captured_at` (if available)
- `heading` (if available)
- `width`, `height`
- `license` (if available)

### **Raw images**
Downloaded images in their original form.  
Rule of thumb: raw images should never be edited, all cleaning happens in processed copies.

### **Stable filename**
A filename format that won’t change between runs, e.g.:
`<source>_<image_id>.jpg`
- Makes it easy to match images to labels and metadata.
- Prevents duplicate downloads and file mismatches.

## Preprocessing terms

### **Preprocessing**
Cleaning and standardizing images so training is consistent.

Common preprocessing steps:
- Remove corrupt/undecodable files
- Fix rotation issues (EXIF)
- Convert to a consistent format (often JPEG)
- Normalize to RGB
- Deduplicate (remove repeated images)

### **EXIF rotation**
Some images are stored “sideways” with a metadata flag that tells viewers how to rotate them.  
“Fix EXIF rotation” means physically rotating the pixel data so the image is upright.

### **RGB**
A standard 3-channel color format (Red/Green/Blue).  
Ensuring RGB avoids training bugs caused by grayscale or CMYK images.

### **Standardize format (JPEG or standardized copies)**
Make images consistent:
- Either convert everything to JPEG, or
- Keep originals and create standardized copies for training

Consistency improves training reliability.

### **Dedup (duplicate removal)**
Removing repeated images.

Types:
- **Exact duplicates:** identical files (often found by hashing).
- **Near duplicates:** visually almost the same image (often detected by perceptual hashing).

### **Hash (exact duplicates)**
A **hash** is a computed fingerprint of a file (e.g., SHA-256/MD5).  
If two files have the same hash, they are identical → can deduplicate safely.

### **Perceptual hash (near duplicates)**
A “visual fingerprint” of an image.  
Two images that look nearly the same can have similar perceptual hashes, even if file bytes differ.

### **Quality flags (`quality_flags`)**
Extra labels in the metadata indicating quality issues like:
- blurry
- too dark
- corrupt
- low resolution

They help you filter problematic images later.

### **`preprocessed_path`**
The file path to the cleaned/standardized version of an image (processed copy).  
Raw image path stays separate.

## Splitting / evaluation terms

### **Train / Val / Test split**
A way to divide images for model development:
- **Train:** used to learn.
- **Validation (val):** used during training to tune settings and detect overfitting.
- **Test:** used at the end for final, unbiased evaluation.

### **Leakage**
When the model sees nearly the same scene in both train and test (e.g., same street corner).  
This makes results look better than real-world performance.

### **Geocell (`geocell_id`)**
A simple way to group images by location:
- You divide the map into a grid (cells)
- Each image belongs to one cell → `geocell_id`
Splitting by geocell helps prevent leakage (same location in multiple splits).

### **Split “by geocell_id” (not random)**
Instead of randomly splitting images, you split by location groups (cells).  
This produces a more honest test of generalization.

## Annotation terms

### **Annotation**
Manually labeling images so the model can learn. For this project you use:
- **Bounding boxes** (rectangles around barriers)
- **Class labels** (stairs, curb/no ramp, broken pavement)

### **Labeling guide (`labeling_guide_v0.1.md`)**
A short rulebook that defines **what counts as each class** and **how to draw boxes**.
Inconsistent labeling is one of the biggest causes of poor model performance.

### **Pilot subset (`pilot_subset.csv`)**
A **small, intentionally chosen set** of images (typically **200–400** here) used for:
- testing the pipeline end-to-end
- initial labeling
- comparing YOLO versions quickly
It avoids spending weeks labeling a huge dataset before you know the approach works.

### **Class map (`class_map.json`)**
A file that maps class names to numeric IDs used in YOLO labels, e.g.:
- `0`: stairs
- `1`: curb_no_ramp
- `2`: broken_pavement
If IDs are inconsistent across exports/training runs, training will be wrong.

### **YOLO label files**
In YOLO format, each image has a `.txt` file with one line per bounding box:
- `class_id x_center y_center width height` (typically normalized 0–1)
You don’t need to memorize the format, but you must ensure exports are consistent.

### **Normalized bounding boxes**
Coordinates are expressed relative to image size (0 to 1), not pixels.  
This makes labels resolution-independent.

## Modeling / experiment terms

### **YOLO baseline**
A “first working model” trained with reasonable default settings, used as a starting point for comparison.

### **YOLO candidates (small/medium/large)**
Different model sizes:
- **small/fast:** faster, less accurate, good for quick iteration
- **medium/balanced:** typical best starting point
- **large/accurate:** better accuracy, slower and needs more compute

### **Experiment protocol (`exp_protocol_v0.md`)**
A document stating fixed settings so comparisons are fair:
- image size
- epochs
- augmentations
- batch size
- dataset split

### **Image size (e.g., 640 or 768)**
The resolution used during training/inference.  
Higher size can help detect small objects (curbs) but costs more compute.

### **Epochs**
One **epoch** = one full pass through the training dataset.  
More epochs can help, but too many can cause overfitting.

### **Augmentations**
Artificial modifications during training to improve robustness:
- brightness/contrast changes
- slight rotation/perspective
- blur/compression

### **Batch size**
How many images the model processes at once during training.  
Larger batch sizes train faster but require more GPU memory.

### **Overfitting**
When a model performs well on training/validation but poorly on new locations.  
Often caused by too little data diversity or leakage.

### **Metrics**
Numbers used to evaluate detection quality.
Common ones in the roadmap:
- **Precision:** of what the model predicted, how many were correct (low precision = many false positives).
- **Recall:** of what actually exists, how many the model found (low recall = many missed detections).
- **mAP@0.5:** a standard object-detection accuracy metric at IoU threshold 0.5 (higher is better).

### **Inference speed**
How fast the model runs (e.g., images per second).  
Important for scaling to many images.

### **Error analysis**
A structured review of typical failures, e.g.:
- broken pavement false positives due to shadows
- curb/no ramp missed when far away or occluded
- stairs missed at side angles

This tells you whether to improve **data**, **labels**, or **model settings**.

### **Saved weights**
The trained model parameters saved to a file so you can reuse the model later without retraining.

### **Comparison table (`results_table_v0.csv`)**
A table that compares YOLO candidates side-by-side (accuracy + speed + size).  
This is what you use to justify “we chose model X”.
