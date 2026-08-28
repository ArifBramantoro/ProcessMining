# Process Mining in CeLOE — revision analysis code

Code accompanying the revised manuscript:

**PONE-D-26-16201**  
*Characterizing learning management system use through process mining: A role-based, cross-faculty analysis of Telkom University’s CeLOE platform*

## Public resources

- Processed dataset: https://figshare.com/articles/dataset/Process_Mining_in_CeLOE/28341992
- Code repository: https://github.com/ArifBramantoro/ProcessMining

The notebooks query the Figshare API directly and download the required CSV files.

## Environment

Recommended runtime: **Python 3.11**.

The analysis pins the process-mining library to:

```text
pm4py==2.7.23.3
```

Each notebook prints the Python, pandas and PM4Py versions at run time. The main
model-quality notebook also saves its analysis settings to
`02_run_metadata.json`; the sensitivity and bootstrap notebooks save their own
metadata files.

Install dependencies with:

```bash
pip install -r requirements.txt
```

## Required data files

For the complete role-based analysis the Figshare deposit should contain the
processed partitions for both roles:

```text
FEB_Student.csv      FEB_Lecturer.csv
FIF_Student.csv      FIF_Lecturer.csv
FIK_Student.csv      FIK_Lecturer.csv
FIT_Student.csv      FIT_Lecturer.csv
FKB_Student.csv      FKB_Lecturer.csv
FRI_Student.csv      FRI_Lecturer.csv
FTE_Student.csv      FTE_Lecturer.csv
```

If a partition is absent, the notebooks report it explicitly and skip analyses
that require that file.

## Analysis window and coverage

The deposited role-partition CSVs are already restricted to records before
**26 June 2023**. Therefore, the deposited files cannot by themselves recover
the percentage of the original all-role export that was retained or discarded
by the date cutoff.

In particular, **do not divide a student-partition count by an all-role raw
export count** to estimate retention. Exact retained/discarded proportions
require the original unfiltered exports or role-matched pre-filter counts.

Notebook `01_baseline_profile.ipynb` reports the contents of the deposited
partitions but deliberately does not manufacture a temporal-retention
percentage.

## Notebook order

### 01 — `01_baseline_profile.ipynb`

Produces:

- event/user/course/trace profile for every available faculty-role partition;
- deduplication counts after the corrected `drop_duplicates()` assignment;
- activity-class shares;
- activity-share heatmap;
- Jensen–Shannon distances and average-linkage faculty clustering.

Outputs include:

```text
01_profile.csv
01_activity_mix.csv
fig_activity_heatmap.png
fig_cluster_student.png
fig_cluster_lecturer.png
```

### 02 — `02_model_quality.ipynb`

Evaluates Alpha, Heuristic and Inductive Miner on:

- token-based fitness;
- ETC token-based precision;
- generalization;
- simplicity;
- Petri-net structural complexity.

It evaluates both user-level and user-course case notions and uses a fixed
random seed for reproducible trace sampling.

The miner variants are explicit:

```text
Alpha Miner       PM4Py Alpha default
Heuristic Miner   Variants.CLASSIC
Inductive Miner   Variants.IM
```

Default evaluation guards:

```text
course-level sample: 300 traces
user-level sample:    40 traces
maximum trace length: 2000 events
random seed:          42
```

Outputs:

```text
02_quality.csv
02_run_metadata.json
```

### 03 — `03_structural_analysis.ipynb`

Computes process-structure evidence that cannot be recovered from raw event
frequency alone:

- directly-follows transitions;
- self-loop burden and leading self-loops;
- quiz attempt revisits;
- **median and p90 inter-view intervals within segmented quiz attempts**;
- A→B→A rework patterns;
- course-level submission-to-next-grading-event latency when both student and
  lecturer partitions are available.

Because the processed data omit the native quiz-attempt key, quiz attempts are
segmented using `attempt_started` boundaries within course-level cases. This is
reported as a measurement limitation.

Outputs:

```text
03_structure.csv
03_quiz_cycle.csv
03_rework_patterns.csv
03_feedback_latency.csv    # only when both role partitions are available
```

### 04 — `04_parameter_sensitivity.ipynb`

Tests whether the default discovery settings materially affect the conclusions.

Heuristic Miner:

```text
variant: Variants.CLASSIC
dependency thresholds: 0.50, 0.70, 0.90, 0.99
```

Inductive Miner sensitivity analysis:

```text
variant: Variants.IMf
noise thresholds: 0.0, 0.1, 0.2, 0.3
```

Note that the **main** Inductive Miner analysis in notebook 02 uses
`Variants.IM`; `Variants.IMf` is used only for the noise-threshold sensitivity
analysis.

Outputs:

```text
04_heuristic_sensitivity.csv
04_inductive_sensitivity.csv
04_run_metadata.json
```

### 05 — `05_statistical_validation.ipynb`

Run notebook 02 first in the same Colab runtime, or place its output at:

```text
/content/02_quality.csv
```

The notebook performs:

- Friedman tests across the three miners;
- Wilcoxon signed-rank pairwise comparisons;
- Holm correction;
- Kendall's W;
- matched-pairs rank-biserial effect sizes;
- trace-level bootstrap confidence intervals.

The bootstrap is a **true with-replacement trace bootstrap**. If an original
trace is sampled more than once, each draw is copied with a unique bootstrap
case identifier. This preserves multiplicity rather than accidentally removing
duplicate draws with `set(...)`.

Bootstrap uncertainty is produced for all four dimensions:

```text
fitness
precision
generalization
simplicity
```

Default bootstrap settings are 100 replicates, 120 traces per replicate,
maximum trace length 2000 and seed base 1000. For final reporting, increase the
number of replicates if compute resources permit and state the final value used.

Outputs:

```text
05_algorithm_stats.csv
05_bootstrap_ci.csv
05_bootstrap_metadata.json
```

## Reproducibility notes

1. Execute notebooks from top to bottom.
2. Do not manually alter the generated CSVs before reporting them.
3. Commit the exact generated result tables used in the manuscript under a
   `results/` directory if possible.
4. An executed version of the notebooks is preferable for peer review because
   the saved outputs expose the exact run-time versions and analysis settings.
5. If you change sample sizes, trace-length limits, seeds, miner variants or
   sensitivity thresholds, update the manuscript methods and reviewer response
   accordingly.

## Code licence

Before resubmission, add an OSI-approved licence to the GitHub repository if
one is not already present. The licence is an author decision and is therefore
not chosen automatically in this package.
