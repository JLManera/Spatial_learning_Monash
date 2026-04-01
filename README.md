# Neuroactive pollution disrupts cognition in fish by causing sex-specific effects on spatial learning

**Published article:** https://pubs.acs.org/doi/10.1021/acs.est.6c00552 (DOI: 10.1021/acs.est.6c00552)

---

## Overview

This repository contains code and data for an experiment investigating how exposure to the antidepressant amitriptyline (at environmentally observed concentrations) alters spatial learning in male and female guppies (*Poecilia reticulata*). Wild-caught fish were exposed for 11 days to either a freshwater control (0 ng L⁻¹), low (≈ 52 ng L⁻¹), or high (≈ 496 ng L⁻¹) concentrations of amitriptyline. After exposure, each individual completed a repeated maze task over twelve trials. Trials were video-recorded, tracked using idtracker.ai, and smoothed using the cleanRfish package. For each trial the time to enter specific maze zones, the number of incorrect decisions, and the total distance travelled were computed.

---

## Running the Code

1. Download this repository and maintain the original folder structure (see below).
2. Open the project file (`Spatial_learning_Monash.Rproj`) in RStudio (or another R-compatible interface).
3. To reproduce the analyses described in the manuscript, run `spatial_cog_data_analysis.Rmd` in its entirety.

> **Note:** Due to storage limitations, the raw tracks used in `spatial_cog_data_wrangling.Rmd` are not provided here. However, the smoothed tracks (CSV files) produced by this script are available in `2-data/2-smoothed_tracks/`, so all steps from the section "5. Summarise Each Trial's Metrics" onward in the data wrangling script can be executed without the raw tracking files.

---

## Folder & File Structure

```
.
├── 1-videos/
│   ├── 1-tomls/              # idtracker.ai configuration files
│   ├── 2-overlaid/           # example clips with tracking overlay for verification
│   └── 3-morph_pics/         # images used for morphometric measurements
├── 2-data/
│   ├── final_df.csv                          # analysis-ready data table (one row per fish-trial)
│   ├── tracking_ROI_and_framerates.csv       # maze region-of-interest coordinates and frame rate metadata
│   ├── treatment_and_sex_data.xlsx           # mapping of each fish to treatment, sex and orientation
│   ├── tracking_start_times.xlsx             # trial start and release times used to align latencies
│   ├── 1-tracks/                             # raw output from idtracker.ai (not included due to size)
│   └── 2-smoothed_tracks/                   # per-trial smoothed XY trajectories
├── 3-water_analysis/
│   └── water_samples.xlsx                   # chemical analysis results from water samples
├── 4-models/                                # saved brms model objects
├── 5-figs/                                  # exported figures from analysis
├── Spatial_learning_Monash.Rproj            # R project file
├── spatial_cog_data_wrangling.Rmd           # data wrangling script
└── spatial_cog_data_analysis.Rmd           # data analysis script
```

---

## Data Documentation

### 1. `final_df.csv`

The primary dataset used for analysis. Contains one row per fish-trial combination. Latencies have been corrected using the release times recorded in `tracking_start_times.xlsx`, so all timing variables reflect the time from actual fish release to first zone entry. Timing variables are in seconds; distances are in centimetres; standard lengths are in millimetres.

| Column | Description |
|--------|-------------|
| `session_name` | Descriptive session name formatted as `camX-bY-dZ-tT_edit_tankN` (camera, batch, day, trial, tank). |
| `ID` | Combination of group and tank (`camX-bY-dZ N`). Used to match fish to calibration and ROI information, which do not change between trials. |
| `trial_round` | Ordinal trial number within a given day (1, 2, or 3). |
| `time_end` | Time (s) when the fish solved the maze (first entry into Correct Zone 2) or reached the 15-minute maximum. |
| `total_distance` | Sum of Euclidean distances between successive positions before `time_end` (cm). |
| `mean_velocity` | Mean swimming speed (cm s⁻¹): `total_distance / time_end`. |
| `incorrect_entries` | Count of incorrect decisions (fish moves from a decision zone into the wrong branch). |
| `time_to_decision_zone_1` | Latency to first entry into Decision Zone 1 (first choice point). `NA` if never entered. |
| `time_to_correct_zone_1` | Latency to first entry into Correct Zone 1. `NA` if not visited. |
| `time_to_incorrect_zone_1` | Latency to first entry into Incorrect Zone 1 (wrong branch, first decision). `NA` if not visited. |
| `time_to_decision_zone_2` | Latency to first entry into Decision Zone 2 (second choice point). `NA` if not visited. |
| `time_to_incorrect_zone_2` | Latency to first entry into Incorrect Zone 2 (wrong branch, second decision). `NA` if not visited. |
| `time_to_correct_zone_2` | Latency to first entry into Correct Zone 2 (goal zone with food reward). Primary solve-time measure. `NA` for censored trials. |
| `group` | Camera–batch–day identifier (e.g., `cam1-b1-d1`). Used to group calibration measurements and release times. |
| `tank` | Tank number (1–4) within the camera view. |
| `frame_rate` | Frame rate of the original video (fps). Most trials were 50 fps; some were 25 or 29.97 fps. |
| `pixels_per_cm` | Group-level calibration factor converting pixel distances to centimetres. |
| `cam` | Camera number. |
| `batch` | Experimental batch number (1–3). |
| `day` | Day of the experiment (1–4). |
| `link` | Concatenation of cam, batch and tank (e.g., `1-1-2`). Used to join behavioural metrics to treatment metadata. |
| `orientation` | Maze orientation: which side the first correct choice is on (`L` or `R`). |
| `jar_ID` | Unique numeric identifier for each fish. Links to treatment and morphometric records. |
| `treatment` | Treatment group: `c` (control), `l` (low ≈ 52 ng L⁻¹), or `h` (high ≈ 496 ng L⁻¹ amitriptyline). |
| `sex` | Biological sex: `M` (male) or `F` (female). |
| `Notes` | Free-form observations (e.g., tracking notes). |
| `std_length` | Standard body length from digital photographs (mm). Included as a covariate. |
| `unique_id` | Camera–batch–day–trial identifier (`camX-bY-dZ-tT`). Four tanks share the same `unique_id`. |
| `tanks3&4_release` | Frame number when tanks 3 and 4 were released. |
| `tanks1&2_release` | Frame number when tanks 1 and 2 were released. |
| `start` | Frame number where the tracking file starts relative to the raw video. Used to calculate latency offsets. |
| `wrong_start` | Tanks (1–4) where fish moved before the trial started. These trials were excluded. `NA` if no issues. |
| `missing_fish` | Tanks (1–4) with absent or untracked fish. These trials were excluded. `NA` if all fish present. |

---

### 2. `treatment_and_sex_data.xlsx`

One row per fish. Links each individual to its treatment group, sex, and maze orientation prior to testing. Sheet "Sheet2" is used by the wrangling code.

| Column | Description |
|--------|-------------|
| `batch` | Batch number (1–3). Matches `batch` in `final_df.csv`. |
| `camera` | Camera number. Matches `cam`. |
| `orientation` | Correct path orientation (`L` or `R`). |
| `tank` | Tank number (1–4). |
| `jar_ID` | Unique fish identifier. Used to link across metadata files. |
| `treatment` | Exposure group: `c`, `l`, or `h`. |
| `sex` | Sex of the fish (`M` or `F`). |
| `Notes` | Free-text notes (e.g., husbandry remarks). |

---

### 3. `tracking_start_times.xlsx`

Provides frame numbers needed to align behavioural latencies with actual release times. In some videos the first few seconds after partition removal contained disturbances that prevented reliable tracking, so initial frames were manually trimmed. The `start` column records the first frame of usable tracking data. Subtracting the appropriate release frame from `start` gives the offset that must be added to all latency measures, ensuring reported latencies represent genuine behavioural responses from the moment of release.

| Column | Description |
|--------|-------------|
| `unique_id` | Camera–batch–day–trial identifier (`camX-bY-dZ-tT`). |
| `tanks3&4_release` | Frame number when tanks 3 and 4 were released. |
| `tanks1&2_release` | Frame number when tanks 1 and 2 were released. |
| `start` | Frame number where tracking begins relative to the raw video. |
| `wrong_start` | Tanks (1–4) where fish started before trial onset. Flagged and excluded. `NA` if none. |
| `missing_fish` | Tanks (1–4) with absent fish. Excluded from analysis. `NA` if none. |
| `notes` | Miscellaneous trial notes. |

---

### 4. `tracking_ROI_and_framerates.csv`

Calibration information and maze region-of-interest (ROI) vertex coordinates for each camera–batch–day combination. Each row represents one vertex. There are 16 vertices per maze, defining the boundaries used to assign positions to decision, correct, and incorrect zones.

| Column | Description |
|--------|-------------|
| `group` | Camera–batch–day identifier (e.g., `cam1-b1-d1`). |
| `cam` | Camera number. |
| `batch` | Batch number. |
| `tank` | Tank number (1–4). |
| `day` | Day of the experiment. |
| `point` | Vertex index (1–16) around the maze perimeter, ordered clockwise. |
| `point_order` | Drawing order of ROI polygons (1–10). Used internally; not used in downstream analyses. |
| `area` | Pixel area of the ROI (unused in analysis). |
| `mean` | Mean pixel intensity within the ROI (unused). |
| `min` | Minimum pixel intensity within the ROI (unused). |
| `max` | Maximum pixel intensity within the ROI (unused). |
| `x` | X-coordinate of the vertex in pixels. |
| `y` | Y-coordinate of the vertex in pixels. |
| `frame_rate` | Frame rate of the video for that group (fps). Typically 50; some groups at 25 or 29.97 fps. |

---

### 5. CSV files in `2-data/2-smoothed_tracks/`

Each CSV corresponds to one fish-trial and contains the smoothed XY trajectory with time stamps. These are intermediates produced by the wrangling script and provided for transparency.

| Column | Description |
|--------|-------------|
| `time` | Elapsed time (s) from the first frame in the tracking file. |
| `x`, `y` | Smoothed coordinates in centimetres (converted from pixels via `pixels_per_cm`). |
| `zone_name` | Assigned maze zone at each time step: `Decision Zone 1`, `Correct Zone 1`, `Incorrect Zone 1`, `Decision Zone 2`, `Incorrect Zone 2`, or `Correct Zone 2`. |
| `ID`, `session_name`, `trial_round`, `frame_rate`, `pixels_per_cm`, `group`, `tank`, `cam`, `batch`, `day` | Metadata linking to summary tables. |
| `track_trim` | Number of frames trimmed at the start to remove pre-release artefacts. Added back when computing latencies. |

---

### 6. Other files

- **`measurements.xlsx`** — morphometric measurements used to compute `std_length`. Contains `jar_ID` and body length (mm) for each fish.
- **`3-water_analysis/water_samples.xlsx`** — chemical analysis results from water samples collected during the experiment, used to verify that amitriptyline concentrations matched nominal levels.

---

## Licensing, Citation & Contact

These materials are released under a [Creative Commons Attribution–NonCommercial–ShareAlike 4.0 International licence (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/).

When using or adapting this code and data, please cite the associated manuscript:

> Manera, J. L., et al. Neuroactive pollution disrupts cognition in fish by causing sex-specific effects on spatial learning. *Environmental Science & Technology*. https://doi.org/10.1021/acs.est.6c00552

For correspondence, contact the first author: **Jack Manera** (jack.manera@monash.edu).
