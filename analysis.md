# CUHK-X Small Model Track — Dataset Analysis Notes

## 1. Project overview

This project is for the **CUHK-X Small Model Track**, a multimodal human activity recognition (HAR) classification task.

The goal is to take an anonymized multimodal test clip and predict one of **40 action classes**, represented by `action_id` values from **0 to 39**.

The dataset uses six modalities:

- `Depth_Color`
- `IR`
- `Thermal`
- `IMU`
- `Radar`
- `Skeleton`

The important idea is that this is **not simply an image-classification problem**. A single action clip can contain multiple sensor streams, and those streams are temporal and may have different sampling rates.

---

## 2. Dataset repository structure

The downloaded project currently has:

```text
Small-Model-Track/
├── README.md
├── Testing/
│   ├── sample_submission.csv
│   ├── test.csv
│   └── small_model_track_test-007/
│       ├── __MACOSX/
│       └── small_model_track_test/
└── Training/
    ├── class_mapping.csv
    └── data/
        ├── Depth_Color/
        ├── IMU/
        ├── IR/
        ├── Radar/
        ├── Skeleton/
        └── Thermal/
```

The extra `__MACOSX` directory contains macOS archive metadata and is not the actual test data.

---

## 3. Test set verification

The actual test data is located at:

```text
Testing/small_model_track_test-007/small_model_track_test/
```

We verified that it contains exactly:

```text
405
```

test clip directories.

They are named:

```text
SM_test_0001
SM_test_0002
...
SM_test_0405
```

This matches the expected test-clip count from the dataset description.

The command used:

```bash
find Testing/small_model_track_test-007/small_model_track_test \
    -maxdepth 1 \
    -type d \
    -name 'SM_test_*' | wc -l
```

Result:

```text
405
```

---

## 4. Test CSV verification

The submission/test CSV is:

```text
Testing/test.csv
```

It contains:

```text
406 lines
```

because there is one header plus 405 test rows.

Example:

```csv
path,prediction
small_model_track_test/SM_test_0001/,
small_model_track_test/SM_test_0002/,
...
```

The test paths run from:

```text
SM_test_0001
```

through:

```text
SM_test_0405
```

Therefore:

- Test clips: **405**
- Test CSV data rows: **405**
- Test CSV total lines: **406**

The `prediction` column is currently empty and will eventually contain an integer from `0` to `39`.

---

## 5. Class mapping

The file:

```text
Training/class_mapping.csv
```

contains **40 classes**.

Full mapping:

| ID | Action |
|---:|---|
| 0 | Wash face |
| 1 | Brush teeth |
| 2 | Comb hair |
| 3 | Take off clothes |
| 4 | Wipe hands |
| 5 | Put on clothes |
| 6 | Drink water |
| 7 | Eat food |
| 8 | Take and use tableware |
| 9 | Pour drinks |
| 10 | Stir drinks |
| 11 | Peel fruits |
| 12 | Sweep the floor |
| 13 | Mop the floor |
| 14 | Wipe bowls |
| 15 | Wipe windows and tables |
| 16 | Fold clothes |
| 17 | Tap the keyboard |
| 18 | Write |
| 19 | Make a phone call |
| 20 | Check the time |
| 21 | Read documents |
| 22 | Turn pages |
| 23 | Listen to music with headphones |
| 24 | Use a mobile phone |
| 25 | Watch TV |
| 26 | Play games |
| 27 | Take a selfie |
| 28 | Jog in place |
| 29 | Do squats |
| 30 | Do jumping jacks |
| 31 | Do stretching exercises |
| 32 | Stand up |
| 33 | Lie down |
| 34 | Sit down |
| 35 | Do lunges |
| 36 | Walk |
| 37 | Take medicine |
| 38 | Massage oneself |
| 39 | Take body temperature |

The actual CSV contains the corresponding names with the numeric prefix, for example:

```text
0,0_Wash_face
1,1_Brush_teeth
...
39,39_Take_body_temperature
```

---

## 6. Training directory structure

The training data follows this general hierarchy:

```text
Training/data/<modality>/<action>/<user>/<trial>/<files>
```

Example:

```text
Training/data/IMU/0_Wash_face/user16/1-1-1/
```

Here:

- `IMU` = modality
- `0_Wash_face` = action/class
- `user16` = subject/user
- `1-1-1` = trial/session identifier

The action folder therefore provides the training label.

---

## 7. Training file counts

We counted files in each training modality.

Current counts:

| Modality | File count |
|---|---:|
| Depth_Color | 85,879 |
| IMU | 5,806 |
| IR | 85,918 |
| Radar | 2,914 |
| Skeleton | 86,050 |
| Thermal | 201,466 |

These are raw file counts, not the number of training clips.

The large difference between modalities is expected because different modalities generate different numbers of files per clip.

---

## 8. Training class distribution

We also inspected how many training trial directories/classes were present for each action.

The important observation is:

> The dataset is not perfectly class-balanced.

For example, several modalities contain approximately:

- `36_Walk`: around 330–335 samples
- `25_Watch_TV`: around 11–12 samples
- many other classes fall between these extremes

Approximate examples from the inspected output:

```text
Depth_Color:
36_Walk                 335
25_Watch_TV              12

IMU:
36_Walk                 330
25_Watch_TV              12

IR:
36_Walk                 335
25_Watch_TV              12

Radar:
36_Walk                 333
25_Watch_TV              12

Skeleton:
36_Walk                 335
25_Watch_TV              12

Thermal:
36_Walk                 324
25_Watch_TV              11
```

This means class imbalance should be considered later when designing the training/validation strategy.

We should **not assume that accuracy alone will tell the complete story during experimentation**, especially for minority classes.

---

## 9. Modalities

The six modalities have different data types.

### 9.1 Depth_Color

Colorized depth frames.

Example filename:

```text
Depth_2025-06-10_10-43-50.016_00000161_Color.png
```

This contains a timestamp and frame/index information.

---

### 9.2 IR

Infrared image frames.

Example:

```text
IR_2025-06-10_10-43-52.616_00000187.png
```

These are grayscale/infrared-style images rather than ordinary RGB images.

---

### 9.3 Thermal

Thermal image frames.

Example:

```text
frame_000410.jpg
```

The thermal stream can contain a different number of frames from the other visual streams.

---

### 9.4 IMU

Inertial measurement data stored in CSV files.

A single trial can contain two CSV files:

```text
down(LL+RL).csv
up(LA+RA+C).csv
```

The exact columns and measurements have **not yet been analyzed**.

---

### 9.5 Radar

Radar data is stored in CSV format.

Example:

```text
radar_output_T2025-06-10_10-43-36.157.csv
```

The exact columns and measurements have **not yet been analyzed**.

---

### 9.6 Skeleton

Skeleton information is stored as JSON files under:

```text
Skeleton/<action>/<user>/<trial>/predictions/
```

Example:

```text
Color_2025-06-10_10-43-49.316_00000154.json
```

The exact JSON structure and skeleton representation have **not yet been analyzed**.

---

## 10. Detailed sample inspection

We inspected one training trial:

```text
0_Wash_face
└── user16
    └── 1-1-1
```

The command showed the following file counts:

| Modality | Files in this trial |
|---|---:|
| Depth_Color | 47 |
| IR | 47 |
| Thermal | 106 |
| IMU | 2 |
| Radar | 1 |
| Skeleton | 47 |

This is an important finding.

A single action trial is represented by multiple synchronized/asynchronous sensor streams rather than a single file.

---

## 11. Example training trial

The sample trial:

```text
Training/data/Depth_Color/0_Wash_face/user16/1-1-1/
```

contains files such as:

```text
Depth_2025-06-10_10-43-50.016_00000161_Color.png
Depth_2025-06-10_10-43-51.216_00000173_Color.png
Depth_2025-06-10_10-43-49.516_00000156_Color.png
...
```

The IR directory contains files such as:

```text
IR_2025-06-10_10-43-52.616_00000187.png
IR_2025-06-10_10-43-50.016_00000161.png
...
```

The Thermal directory contains:

```text
frame_000410.jpg
frame_000434.jpg
frame_000449.jpg
...
```

The IMU directory contains:

```text
down(LL+RL).csv
up(LA+RA+C).csv
```

The Radar directory contains one CSV:

```text
radar_output_T2025-06-10_10-43-36.157.csv
```

The Skeleton directory contains many JSON prediction files with timestamp/frame information.

---

## 12. Important observation: different sampling rates

The sample trial demonstrates that the modalities do not necessarily contain the same number of frames:

```text
Depth_Color   47
IR            47
Skeleton      47
Thermal       106
IMU            2 CSV files
Radar          1 CSV file
```

Therefore, we should **not assume** a simple one-to-one mapping such as:

```text
Depth frame 1
IR frame 1
Thermal frame 1
Skeleton frame 1
IMU frame 1
Radar frame 1
```

Instead, the timestamps, sampling rates, and temporal alignment need to be investigated.

This will likely be an important part of the eventual multimodal model.

---

## 13. Test sample inspection

The test clip:

```text
Testing/.../small_model_track_test/SM_test_0001/
```

contains the same six modality directories:

```text
Depth_Color/
IMU/
IR/
Radar/
Skeleton/
Thermal/
```

The test data is anonymized: the action label is not included in the directory name.

For example:

```text
SM_test_0001
```

does not directly reveal the action.

The model must infer the action and produce an integer class ID.

---

## 14. Test dataset verification issue resolved

An earlier recursive search reported:

```text
407
```

directories because the extracted archive also contains macOS metadata under:

```text
__MACOSX/
```

and other non-data paths.

After restricting the search to the actual test root:

```text
Testing/small_model_track_test-007/small_model_track_test/
```

the correct count is:

```text
405
```

This matches `test.csv`.

Therefore the actual test set appears structurally consistent.

---

# 15. What has been completed

### Dataset understanding

- [x] Confirmed repository structure
- [x] Located Training data
- [x] Located Testing data
- [x] Confirmed 405 actual test clips
- [x] Confirmed 405 test CSV rows
- [x] Confirmed 40 classes
- [x] Read class mapping
- [x] Identified six modalities
- [x] Counted training files by modality
- [x] Inspected training class distribution
- [x] Inspected one complete multimodal training trial
- [x] Confirmed different modalities have different file counts
- [x] Confirmed test clips contain the six modality directories

---

# 16. What remains to analyze

We are now moving from **dataset bookkeeping** into actual data analysis.

The next things to investigate are:

### A. IMU

Determine:

- CSV columns
- sensor dimensions
- number of rows
- timestamps
- sampling frequency
- accelerometer/gyroscope information
- whether the two CSVs represent different body locations/sensors

### B. Radar

Determine:

- CSV columns
- timestamps
- number of measurements
- radar feature structure
- sampling frequency
- whether measurements are already processed

### C. Skeleton

Determine:

- JSON schema
- number of joints
- coordinates
- confidence values
- timestamps
- whether skeletons are 2D or 3D
- how missing detections are represented

### D. Images

Determine:

- image dimensions
- channels
- pixel/value ranges
- temporal ordering
- timestamps
- differences between Depth_Color, IR and Thermal

### E. Temporal alignment

Determine how:

```text
Depth_Color
IR
Thermal
Skeleton
IMU
Radar
```

relate to one another in time.

### F. Missing modalities

Check whether every clip contains every modality and whether some modalities are incomplete.

### G. Train/validation split

Because the dataset contains users and trials, we need to determine whether splitting randomly could cause subject leakage.

A subject-aware validation strategy may be important.

### H. Baseline

Only after the above:

```text
single modality baseline
        ↓
strong baseline
        ↓
multimodal fusion
        ↓
optimization
        ↓
submission
```

---

# 17. Current strategy

We should **not start with the biggest possible model**.

The intended progression is:

```text
UNDERSTAND DATA
      ↓
CREATE VALIDATION SPLIT
      ↓
BUILD SIMPLE BASELINE
      ↓
MEASURE WHICH MODALITIES HELP
      ↓
TEMPORAL MODEL
      ↓
MULTIMODAL FUSION
      ↓
OPTIMIZE
      ↓
FINAL TEST PREDICTIONS
```

The laptop does not need to train the final model.

The local machine can be used for:

- data inspection
- preprocessing development
- small experiments
- debugging
- dataset organization

GPU training can be moved to an external GPU environment later.

---

# 18. Important current conclusion

At this point, there is **no evidence that the downloaded dataset is missing its core training/test files**.

The verified structure is consistent with a multimodal HAR dataset:

```text
405 test clips
40 action classes
6 modalities
large multimodal training set
different sampling rates/file counts
```
