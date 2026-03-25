# Experimental Data and Scripts

This directory contains the raw experimental data, batch execution scripts, and data analysis scripts for the evaluation section of the paper. The experiments were conducted on the SPECVER25 benchmark dataset to evaluate the practical applicability and classification effectiveness of the proposed best-effort GR(1) synthesis approach.

## 1 Research Questions

- **RQ1**: Do best-effort specifications occur in the SYNTECH dataset? If so, what is their distribution?
- **RQ2**: Do the three categories defined for quantifying the degree of unrealizability occur among well-separated yet unrealizable specifications? If so, what is their distribution?

## 2 Setting Up the Experimental Environment

First, download the Docker image `best-effort-gr1.tar` from the **Assets** section of the [GitHub Release](https://github.com/besteffortgr1/Supplementary-Material-and-Artifact/releases/tag/v1) page (the file is too large to include in the repository).

Load the Docker image:

```bash
docker load -i best-effort-gr1.tar
```

Start the container:

```bash
docker run -it best-effort-gr1:latest /bin/bash
```

Inside the container, in the `/workspace` directory, copy the BDD library to the system library path (required):

```bash
cp libcudd.so /usr/lib
```

## 3 Directory Contents

| File | Description |
|------|-------------|
| `SPECVER25_result1.csv` | Phase 1 raw results (RQ1) |
| `SPECVER25_result2.csv` | Phase 2 raw results (RQ2) |
| `bestEffortCheck.jar` | Spectra GR(1) synthesizer executable |
| `libcudd.so` | Modified CUDD BDD library required by the synthesizer |
| `result1_batch_run.py` | Phase 1 batch execution script |
| `result2_batch_run.py` | Phase 2 batch execution script |
| `result1_analysis.py` | RQ1 data analysis script |
| `result2_analysis.py` | RQ2 data analysis script |
| `dataset` | SPECVER25 dataset |

## 4 Experimental Procedure

### 4.1 Phase 1: Best-Effort Realizability Check (RQ1)

**Script**: `result1_batch_run.py`

This script recursively traverses all `.spectra` files in the SPECVER25 dataset and invokes `bestEffortCheck.jar` to perform best-effort realizability checking (with the `-bec` flag). Each specification is subject to a 600-second timeout. Results are appended to a CSV file; for specifications that time out or terminate abnormally, the corresponding status markers are written by the Python script.

**Usage**:

```bash
python result1_batch_run.py --spectra-dir dataset --jar bestEffortCheck.jar --output local_result1.csv
```

Note: Use a different output filename (e.g., `local_result1.csv`) to avoid overwriting the existing raw data file `SPECVER25_result1.csv`.


**Output**: `SPECVER25_result1.csv`

### 4.2 Phase 2: Three-Category Classification (RQ2)

**Script**: `result2_batch_run.py`

This script filters specifications from the Phase 1 results that satisfy both of the following conditions: standard GR(1) realizability result is `Unrealizable`, and well-separatedness result is `Well-Separated`. For each filtered specification, it performs best-effort checking with classification (using the `-bec -c` flags) without a timeout limit. The script supports checkpointing; it automatically detects previously processed files before execution to avoid redundant work.



**Usage**:

```bash
python result2_batch_run.py --input local_result1.csv --jar bestEffortCheck.jar --output local_result2.csv
```

Note: Use a different output filename (e.g., `local_result2.csv`) to avoid overwriting the existing raw data file `SPECVER25_result2.csv`.

**Output**: `SPECVER25_result2.csv`

## 5 Data File Format

### 5.1 SPECVER25_result1.csv

This file contains the Phase 1 experimental results, with a header line `SpectraFile,Result`. Valid data rows contain 4 comma-separated fields:

| Column | Field | Values |
|--------|-------|--------|
| 1 | File path | Relative path to the `.spectra` file |
| 2 | Best-effort realizability result | `Realizable` or `Unrealizable` |
| 3 | Standard GR(1) realizability result | `Realizable` or `Unrealizable` |
| 4 | Well-separatedness | `Well-Separated` or `Non-Well-Separated` |

Rows with fewer than 4 fields are non-valid entries that may contain status markers such as `TIMEOUT`, `UNKNOWN`, or `ERROR`, and are excluded from the analysis.

The file contains 10,137 data rows (plus 1 header row), of which 8,249 are valid records and 1,888 are non-valid entries (1,284 timeouts, 380 unknown, 224 errors).

### 5.2 SPECVER25_result2.csv

This file contains the Phase 2 experimental results, with no header line. Each row contains 5 comma-separated fields:

| Column | Field | Values |
|--------|-------|--------|
| 1 | File path | Relative path to the `.spectra` file |
| 2 | Best-effort realizability result | `Realizable` (BE-realizable) or `Unrealizable` (Trapped) |
| 3 | Standard GR(1) realizability result | All rows: `Unrealizable` |
| 4 | Well-separatedness | All rows: `Well-Separated` |
| 5 | Algorithm category | `Expand` or `Construct` |

The file contains 2,081 rows, corresponding to all well-separated and unrealizable specifications from Phase 1.

**Mapping to paper categories** (Table 2):

| Paper Category | Definition | CSV Condition |
|----------------|------------|---------------|
| Extended | Z_std ≠ ∅ | Column 2 = `Realizable` AND Column 5 = `Expand` |
| Constructed | Z_std = ∅, ι ⊆ Z_BE | Column 2 = `Realizable` AND Column 5 = `Construct` |
| Trapped | ι ⊄ Z_BE | Column 2 = `Unrealizable` |

## 6 Data Analysis Scripts

### 6.1 result1_analysis.py (RQ1 Data Analysis)

This script parses `SPECVER25_result1.csv` and computes the statistics required for Table 1 (RQ1), including: total number of valid specifications, number of realizable and unrealizable specifications, well-separated and non-well-separated breakdown of unrealizable specifications, and best-effort realizable and not best-effort realizable counts. The script outputs a formatted statistics table, LaTeX-ready table rows, and narrative text drafts.

**Usage**:

```bash
python result1_analysis.py SPECVER25_result1.csv
```

### 6.2 result2_analysis.py (RQ2 Data Analysis)

This script parses `SPECVER25_result2.csv` and computes the three-category classification statistics required for Table 2 (RQ2). The classification rules are as follows:

| Category | Condition | Meaning |
|----------|-----------|---------|
| Extended | BE-realizable AND Expand | Standard winning region is non-empty; best-effort winning region extends it |
| Constructed | BE-realizable AND Construct | Standard winning region is empty; best-effort winning region is constructed from scratch |
| Trapped | BE-unrealizable | Initial states are outside the best-effort winning region; environment has an absolute counter-strategy |

**Usage**:

```bash
python result2_analysis.py SPECVER25_result2.csv
```

## 7 Results Summary

### RQ1 Results (Table 1)

| Category | Count | Percentage |
|----------|-------|------------|
| Valid specifications | 8,249 | 100% |
| Realizable | 5,251 | 63.7% of valid |
| Unrealizable | 2,998 | 36.3% of valid |
| Non-well-separated | 917 | 30.6% of unrealizable |
| Well-separated | 2,081 | 69.4% of unrealizable |
| BE-realizable | 1,066 | 51.2% of well-separated |
| Not BE-realizable | 1,015 | 48.8% of well-separated |

### RQ2 Results (Table 2)

| Category | Count | Percentage |
|----------|-------|------------|
| Well-separated & unrealizable (total) | 2,081 | 100% |
| Extended (Z_std ≠ ∅) | 936 | 45.0% |
| Constructed (Z_std = ∅, ι ⊆ Z_BE) | 130 | 6.2% |
| Trapped (ι ⊄ Z_BE) | 1,015 | 48.8% |
