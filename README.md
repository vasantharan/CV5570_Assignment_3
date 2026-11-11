
# CV5570 - Computer Vision for Construction Engineering

## Assignment 3: Stereo Visual SLAM

---

## Student Details

- **Name:** Vasantharan K
- **Roll No:** ID26S012
- **Program:** M.S
- **Department:** School of Interdisciplinary Studies
- **Course:** CV5570 - Computer Vision for Construction Engineering
- **Semester:** Jul - Nov 2026

---

# 1. Assignment Overview

This assignment implements important components of a **Stereo Visual SLAM pipeline** using the **KITTI Visual Odometry dataset**.

The work focuses on:

- Stereo image processing
- Disparity estimation
- Stereo depth estimation
- ORB feature extraction and matching
- Stereo Visual Odometry
- Camera trajectory estimation
- Absolute Trajectory Error (ATE) evaluation
- Keyframe selection
- Loop closure candidate detection
- Geometric verification of loop closures

The implementation was tested using **KITTI Odometry Sequence 05**.

---

# 2. Objectives

The main objectives of this assignment are:

1. Load and process stereo image sequences from the KITTI dataset.
2. Compute a disparity map from the left and right stereo images.
3. Estimate scene depth using stereo geometry.
4. Extract and match visual features using ORB.
5. Estimate camera motion between consecutive frames.
6. Construct the Visual Odometry trajectory.
7. Compare the estimated trajectory with KITTI ground truth.
8. Evaluate trajectory accuracy using Absolute Trajectory Error (ATE).
9. Select keyframes for efficient loop closure detection.
10. Detect potential loop closure candidates.
11. Verify loop closure candidates using geometric consistency.

---

# 3. Dataset

The assignment uses the **KITTI Visual Odometry Dataset**, specifically:

- **Sequence:** 05
- **Stereo Images:** `image_0` and `image_1`
- **Calibration:** `calib.txt`
- **Timestamps:** `times.txt`
- **Ground Truth Poses:** `poses/05.txt`

The KITTI dataset provides stereo image sequences and ground-truth trajectories for evaluating Visual Odometry algorithms.

---

# 4. Project Structure

```text
CV5570_Assignment_3/
│
├── Assignment_3.ipynb
├── requirements.txt
├── README.md
│
├── data/
│   ├── sequences/
│   │   └── 05/
│   │       ├── image_0/
│   │       ├── image_1/
│   │       ├── calib.txt
│   │       └── times.txt
│   │
│   └── poses/
│       └── 05.txt
│
├── output/
│   ├── disparity/
│   ├── depth/
│   ├── plots/
│   │   ├── stereo_disparity_depth_000000.png
│   │   ├── gt_vs_vo_trajectory.png
│   │   ├── vo_vs_ground_truth.png
│   │   ├── loop_similarity_matrix.png
│   │   ├── accepted_loop.png
│   │   └── rejected_loop.png
│   │
│   ├── results_summary.csv
│   └── vo_05.txt
│
└── report/
    └── Assignment_3_Report.pdf
```

---

# 5. Environment Setup

## Create and Activate a Virtual Environment

```powershell
python -m venv env
```

Activate the environment:

```powershell
.\env\Scripts\Activate.ps1
```

---

## Install Required Packages

```powershell
pip install numpy opencv-python matplotlib pandas scipy jupyter
```

For trajectory evaluation:

```powershell
pip install evo
```

Additional packages may be installed using:

```powershell
pip install -r requirements.txt
```

---

# 6. Task 2 – Stereo Visual Odometry

## 6.1 Stereo Image Loading

The left and right stereo images are loaded from:

```text
data/sequences/05/image_0/
data/sequences/05/image_1/
```

The stereo pair provides two views of the same scene from cameras separated by a known baseline.

---

## 6.2 Disparity Estimation

Disparity represents the horizontal displacement of corresponding points between the left and right stereo images.

The disparity is calculated as:

\[
d = x_L - x_R
\]

where:

- \(x_L\) is the horizontal coordinate in the left image.
- \(x_R\) is the horizontal coordinate in the right image.
- \(d\) is the disparity.

The computed disparity map has the following shape:

```text
Disparity shape: (370, 1226)
```

Observed disparity range:

```text
Minimum disparity: -1.0
Maximum disparity: 78.25
```

---

## 6.3 Depth Estimation

Depth is estimated using the stereo geometry relationship:

\[
Z = \frac{fB}{d}
\]

where:

- \(Z\) = depth
- \(f\) = camera focal length
- \(B\) = stereo camera baseline
- \(d\) = disparity

The valid depth values obtained were:

```text
Minimum valid depth: 4.8538594 metres
Maximum valid depth: 675.22577 metres
```

### Output

The stereo images, disparity map, and depth map are visualized in:

```text
output/plots/stereo_disparity_depth_000000.png
```

---

# 7. Stereo Feature Matching

ORB (**Oriented FAST and Rotated BRIEF**) features are used to identify corresponding visual features between stereo images.

The general process is:

1. Detect ORB keypoints.
2. Compute ORB descriptors.
3. Match descriptors between the left and right images.
4. Apply matching criteria to retain reliable correspondences.
5. Use stereo correspondences for 3D information.

ORB was selected because it is computationally efficient and suitable for real-time Visual Odometry applications.

---

# 8. Visual Odometry

Visual Odometry estimates the motion of a camera by analyzing consecutive images.

For each frame pair, the pipeline performs:

1. Feature detection.
2. Feature matching.
3. 3D point estimation from stereo depth.
4. 2D–3D correspondence generation.
5. Camera pose estimation using geometric methods.
6. Incremental trajectory estimation.

The estimated poses are accumulated to construct the complete camera trajectory.

The generated Visual Odometry trajectory is saved as:

```text
output/vo_05.txt
```

---

# 9. Visual Odometry vs Ground Truth

The estimated trajectory is compared against the KITTI ground-truth trajectory.

The ground truth is located at:

```text
data/poses/05.txt
```

The estimated Visual Odometry trajectory is:

```text
output/vo_05.txt
```

The following plots are generated:

```text
output/plots/gt_vs_vo_trajectory.png
```

and

```text
output/plots/vo_vs_ground_truth.png
```

These plots visually compare the estimated Visual Odometry trajectory with the KITTI ground-truth trajectory.

---

# 10. Trajectory Evaluation

Trajectory accuracy is evaluated using the `evo` toolkit.

The command used is:

```powershell
evo_ape kitti data\poses\05.txt output\vo_05.txt -a --plot --plot_mode xz
```

The resulting Absolute Trajectory Error (ATE) statistics are:

```text
APE w.r.t. translation part (m)
(with SE(3) Umeyama alignment)

max      22.334345
mean      8.626315
median    8.167589
min       0.173949
rmse      9.621870
sse    255614.452965
std       4.262286
```

## Final ATE Result

### **ATE RMSE = 9.621870 metres**

The error generally increases as the camera travels further because small pose estimation errors accumulate over time. This accumulation is commonly referred to as **drift** in Visual Odometry.

---

# 11. Visual SLAM vs Visual Odometry

## Visual Odometry (VO)

Visual Odometry estimates the camera's motion by tracking visual information between consecutive frames.

The main objective is to estimate:

- Camera position
- Camera orientation
- Camera trajectory

However, small errors accumulate over time, causing trajectory drift.

---

## Visual SLAM

Visual SLAM performs:

- Camera localisation
- Mapping of the environment
- Loop closure detection
- Global trajectory correction

Unlike Visual Odometry, Visual SLAM can use loop closures and global optimisation to reduce accumulated drift.

### Main Difference

| Visual Odometry | Visual SLAM |
|---|---|
| Estimates camera motion | Estimates camera motion and builds a map |
| Uses mainly sequential observations | Can use previous observations |
| Errors accumulate over time | Loop closures can reduce drift |
| No global correction by default | Can perform global optimisation |

---

# 12. Task 3 – Loop Closure Detection and Geometric Verification

Loop closure occurs when the camera revisits a previously observed location.

Detecting loop closures is important because they can potentially help correct accumulated trajectory drift.

---

## 12.1 Keyframe Selection

Instead of processing every frame for loop closure detection, representative frames are selected as **keyframes**.

Keyframe selection reduces:

- Computational cost
- Number of unnecessary comparisons
- Memory requirements

The selected keyframes are then compared to identify visually similar locations.

---

## 12.2 Loop Closure Candidate Detection

Visual similarity between keyframes is evaluated using feature correspondences.

A similarity matrix is generated and visualized at:

```text
output/plots/loop_similarity_matrix.png
```

Potential loop closure candidates are identified from keyframes that show high visual similarity.

---

## 12.3 Geometric Verification

Visual similarity alone is not sufficient to accept a loop closure.

Two images may have many matching features but still represent different physical locations.

Therefore, candidate loop closures undergo geometric verification.

The verification process checks whether the matched features are geometrically consistent.

This helps reject false loop closures caused by:

- Repeated structures
- Similar buildings
- Repeated columns
- Similar floors
- Construction environments with repetitive patterns

---

## 12.4 Accepted Loop Closure

An example of a successfully verified loop closure is shown in:

```text
output/plots/accepted_loop.png
```

The candidate passed the geometric verification stage.

---

## 12.5 Rejected Loop Closure

An example of a rejected loop closure is shown in:

```text
output/plots/rejected_loop.png
```

This demonstrates why simply counting feature matches is not sufficient.

A candidate can have visually similar features but fail the geometric consistency test.

---

# 13. Loop Closure Results

The implementation produced the following results:

| Result | Value |
|---|---:|
| Selected Keyframes | 407 |
| Loop Candidates | 321 |
| Accepted Loops | 307 |
| Rejected Loops | 14 |

Geometric verification is essential because repetitive structures may produce false feature correspondences.

For example, on a construction site, visually similar floors or repeated columns can create many feature matches even when the camera is observing different locations.

---

# 14. Task 4 – Pose Graph Optimisation and Trajectory Correction (Bonus)

The bonus task involved performing global pose graph optimisation.

The intended approach was to:

1. Create nodes representing camera poses.
2. Add odometry constraints between consecutive poses.
3. Add verified loop closure constraints.
4. Optimise the pose graph.
5. Compare the trajectory before and after optimisation.
6. Evaluate the reduction in trajectory error.
7. Test the effect of an intentionally incorrect loop closure.

The intended implementation used **GTSAM** for pose graph optimisation.

However, the required GTSAM Python package could not be successfully installed in the available Python/Windows environment.

The installation attempt:

```powershell
pip install gtsam
```

resulted in:

```text
ERROR: Could not find a version that satisfies the requirement gtsam
ERROR: No matching distribution found for gtsam
```

Due to the environment compatibility issue and available time constraints, the bonus task was not completed.

Therefore, the following bonus results are **not reported**:

- ATE before optimisation
- ATE after optimisation
- Reduction in ATE after loop closures
- Global trajectory correction
- Intentionally incorrect loop closure experiment

No experimental values were fabricated for the skipped bonus task.

---

# 15. Results Summary

The generated output files include:

```text
output/
├── disparity/
├── depth/
├── plots/
│   ├── accepted_loop.png
│   ├── gt_vs_vo_trajectory.png
│   ├── loop_similarity_matrix.png
│   ├── rejected_loop.png
│   ├── stereo_disparity_depth_000000.png
│   └── vo_vs_ground_truth.png
│
├── results_summary.csv
└── vo_05.txt
```

### Main Results

- Stereo disparity successfully computed.
- Stereo depth successfully estimated.
- Stereo Visual Odometry trajectory generated.
- Ground truth comparison performed.
- **ATE RMSE obtained: 9.621870 metres.**
- Loop closure candidates detected.
- Loop candidates geometrically verified.
- Accepted and rejected loop examples generated.
- Pose graph optimisation bonus task skipped due to the GTSAM Python environment installation issue.

---

# 16. Output Visualisations

## Stereo Disparity and Depth

```text
output/plots/stereo_disparity_depth_000000.png
```

This figure contains:

- Left stereo image
- Right stereo image
- Disparity map
- Depth map

---

## Visual Odometry Trajectory

```text
output/plots/gt_vs_vo_trajectory.png
```

and

```text
output/plots/vo_vs_ground_truth.png
```

These figures compare the estimated trajectory with the KITTI ground truth.

---

## Loop Closure Similarity Matrix

```text
output/plots/loop_similarity_matrix.png
```

This visualizes the similarity between selected keyframes.

---

## Accepted and Rejected Loop Closures

```text
output/plots/accepted_loop.png
```

```text
output/plots/rejected_loop.png
```

These figures demonstrate the results of geometric loop verification