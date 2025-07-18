# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

This project uses a Python virtual environment located at `dataset-venv/`. To activate it:

```bash
source dataset-venv/bin/activate
```

Key installed packages include TensorFlow 2.19.0, PyTorch 2.7.1, scikit-learn 1.6.1, pandas 2.2.3, and Keras 3.10.0.

## Project Architecture

This is a machine learning research project for cybersecurity dataset evaluation using anomaly detection models. The project follows a structured pipeline approach with three main datasets:

### Recent Work (Latest Session)

**Completed Work on APT Dataset Analysis:**
- **Network Traffic Flow Analysis**: Created comprehensive exploratory and structure consistency analysis for network flow data
  - `3a-exploratory_network-traffic-flow-index.ipynb`: Exploratory analysis with organized logging (ANALYSIS_TYPE = "3a-network-flows")
  - `3b-structure-consistency-analyzer.ipynb`: Structure pattern detection and schema analysis
  - `3_elastic_network-traffic-flow-ds_csv_creator.ipynb`: Enhanced CSV converter with documentation and array handling fixes
- **Sysmon Events Analysis**: Created parallel analysis notebooks for Windows Sysmon events
  - `2a-exploratory_sysmon-index.ipynb`: Sysmon EventID distribution and XML structure analysis  
  - `2b-structure-consistency-analyzer.ipynb`: Sysmon structure consistency and field pattern analysis
- **Key Improvements Made**:
  - Fixed port number data types (float → integer conversion in CSV creator)
  - Added comprehensive exploratory data analysis with 7 analysis sections covering security patterns
  - Implemented robust NaN handling for IP addresses and missing data
  - Created organized output directory structure (`outputs/3a-network-flows/`, `outputs/3b-network-flows/`, etc.)
  - Enhanced logging system with timestamped analysis reports

**Current Data Processing Status:**
- Network flow JSONL: Fully analyzed and optimized CSV conversion pipeline ready
- Sysmon JSONL: Analysis notebooks created, ready for execution and optimization
- All notebooks include comprehensive documentation and error handling

### Folder Structure

```
research/
├── datasets/                    # Raw cybersecurity datasets
│   ├── bigbase/                # Windows event logs (50 CSV files)
│   ├── unraveled/              # Multi-modal cybersecurity data
│   └── dapt2020/               # Network flow data (CICFlowMeter)
├── analysis/                   # ML pipeline scripts and configurations
│   ├── sae/                    # Stacked Autoencoder experiments
│   ├── lstm-sae/               # LSTM-SAE experiments
│   ├── iforest/                # Isolation Forest experiments
│   ├── gru-sae/                # GRU-SAE experiments
│   ├── config/                 # Schema and encoding configurations
│   ├── utils/                  # Shared utilities (paths, encoding, config)
│   └── comparative-evaluation/ # Cross-model result analysis
├── data/                       # Pipeline output sinks
│   ├── processed/              # Aggregated datasets
│   └── encoded/                # Feature-engineered data
├── models/                     # Trained model artifacts
├── artifacts/                  # Analysis outputs
│   ├── eval/                   # Model evaluation results
│   ├── plots/                  # Visualizations
│   ├── metadata/               # Dataset metadata
│   ├── comparative/            # Cross-model analysis
│   └── quality_analysis/       # Dataset quality reports
├── docs/                       # Documentation
├── pdfs/                       # Research papers and reports
├── others/                     # Miscellaneous files
└── dataset-venv/               # Python virtual environment
```

### Datasets Structure
- **datasets/bigbase/**: 50 CSV files (dataset-01.csv to dataset-50.csv) with Windows event logs containing TF-IDF processable text columns (CommandLine, ParentCommandLine, Image paths)
- **datasets/unraveled/**: Multi-modal cybersecurity data including host logs (audit, auth, filebeat, syslog, windows) and network flows organized by weeks/days
- **datasets/dapt2020/**: Network flow data with 85 features extracted by CICFlowMeter, including 76 flow features plus Activity/Stage labels

### Analysis Pipeline Architecture

The analysis follows a standardized **5-stage pipeline** organized by model type in separate directories:

**Model Directories:**
- `analysis/sae/` - Stacked Autoencoder experiments
- `analysis/lstm-sae/` - LSTM-based Stacked Autoencoder experiments  
- `analysis/iforest/` - Isolation Forest experiments
- `analysis/gru-sae/` - GRU-based Stacked Autoencoder experiments

**Pipeline Stages (each model directory contains):**
1. **Stage 01: Data Aggregation** (`01-aggregation.py`)
   - Combines raw CSV files using schema configurations
   - Maps labels to binary (0=Benign, 1=Attack) 
   - Outputs: processed parquet, label distribution, metadata

2. **Stage 02: Data Encoding** (model-specific encoding)
   - `sae/`: `02-encoding-bigbase-sae.py`, `02-encoding-unraveled-sae.py`
   - `lstm-sae/`: `02-encoding-lstm-sae.py` (creates 3D sequences for temporal modeling)
   - `iforest/`: `02-encoding-bigbase-iforest.py`, `02-encoding-unraveled-iforest.py`
   - `gru-sae/`: `02-encoding-gru-sae.py` (creates 3D sequences for temporal modeling)
   - Applies transformations: TF-IDF + OneHot + StandardScaler (bigbase), OneHot + StandardScaler (unraveled)

3. **Stage 03: Model Training** (model-specific training)
   - `sae/`: `03-training-sae.py` - Stacked autoencoder on 2D features
   - `lstm-sae/`: `03-training-lstm-sae.py` - LSTM autoencoder on 3D sequences
   - `iforest/`: `03-training-iforest.py` - Isolation Forest with PCA dimensionality reduction
   - `gru-sae/`: `03-training-gru-sae.py` - GRU autoencoder on 3D sequences
   - All models train on benign data only for anomaly detection

4. **Stage 04: Model Evaluation** (`04-model-evaluation-*.py`)
   - Computes model-specific anomaly scores (reconstruction error, isolation score)
   - Evaluates on test sets with ROC-AUC, precision, recall metrics

5. **Stage 05: Result Visualization** (`05-result-vis*.py`)
   - `sae/`: `05-result-vis.py` 
   - `lstm-sae/`: `05-result-vis-lstm-sae.py`
   - `iforest/`: `05-feature-space-iforest.py` (feature space analysis)
   - `gru-sae/`: `05-result-vis-gru-sae.py`

### Configuration System

Experiments use JSON configurations in `analysis/config/`:

**Schema configs** (`schema-*.json`): Define core features, label columns, benign labels
- `schema-bigbase-v1.json`, `schema-bigbase-v2.json` - Windows event log schemas
- `schema-unraveled-v1.json` - Network flow schema

**Encoding configs** (`encoding-*.json`): Specify feature transformations per model/dataset
- `encoding-bigbase-v1.json` - Basic bigbase TF-IDF + OneHot encoding
- `encoding-bigbase-v2.json` - Extended bigbase with additional TF-IDF columns (OriginalFileName, TargetObject, TargetFilename, PipeName)  
- `encoding-unraveled-sae-v1.json` - Unraveled network features encoding
- `encoding-unraveled-lstm-v1.json` - Unraveled encoding with time column for LSTM sequences

### Path Management

The `analysis/utils/paths.py` module provides centralized path management with functions like:
- `get_processed_path(model, dataset, version)`: Processed data location in `data/processed/`
- `get_encoded_dir(model, dataset, version)`: Encoded features directory in `data/encoded/`
- `get_model_dir(model, dataset, version)`: Trained model storage in `models/`
- `get_eval_dir(model, dataset, version)`: Evaluation results in `artifacts/eval/`

### Artifact Storage

- **data/processed/**: Aggregated and cleaned datasets from Stage 1
- **data/encoded/**: Feature-engineered datasets from Stage 2
- **models/**: Trained model files from Stage 3
- **artifacts/eval/**: Evaluation metrics and results from Stage 4
- **artifacts/plots/**: Visualizations from Stage 5
- **artifacts/comparative/**: Cross-model comparison results
- **artifacts/quality_analysis/**: Dataset quality analysis reports

## Common Commands

### SAE (Stacked Autoencoder) Pipeline
```bash
# Stage 1: Data aggregation
python analysis/sae/01-aggregation.py --model sae --dataset bigbase --version v1 --schema analysis/config/schema-bigbase-sae-v1.json

# Stage 2: Encoding 
python analysis/sae/02-encoding-bigbase-sae.py --dataset bigbase --version v1 --model sae --config analysis/config/encoding-bigbase-sae-v1.json

# Stage 3: Training
python analysis/sae/03-training-sae.py --dataset bigbase --version v1 --model sae

# Stage 4: Evaluation
python analysis/sae/04-model-evaluation-sae.py --dataset bigbase --version v1 --model sae

# Stage 5: Visualization
python analysis/sae/05-result-vis.py --dataset bigbase --version v1 --model sae
```

### LSTM-SAE (LSTM Autoencoder) Pipeline
```bash
# Stage 1: Data aggregation
python analysis/lstm-sae/01-aggregation.py --model lstm-sae --dataset unraveled --version v1 --schema analysis/config/schema-unraveled-lstm-sae-v1.json

# Stage 2: LSTM-specific sequence encoding
python analysis/lstm-sae/02-encoding-lstm-sae.py --dataset unraveled --version v1 --model lstm-sae --encoding_config analysis/config/encoding-unraveled-lstm-sae-v1.json --seq-len 10 --sample_size 20000

# Stage 3: LSTM-SAE Training
python analysis/lstm-sae/03-training-lstm-sae.py --dataset unraveled --version v1 --model lstm-sae --batch-size 16 --epochs 10

# Stage 4: LSTM-SAE Evaluation  
python analysis/lstm-sae/04-model-evaluation-lstm-sae.py --dataset unraveled --version v1 --model lstm-sae

# Stage 5: LSTM-specific visualization
python analysis/lstm-sae/05-result-vis-lstm-sae.py --dataset unraveled --version v1 --model lstm-sae
```

### Isolation Forest Pipeline
```bash
# Stage 1: Data aggregation
python analysis/iforest/01-aggregation.py --model iforest --dataset bigbase --version v1 --schema analysis/config/schema-bigbase-iforest-v1.json

# Stage 2: Encoding 
python analysis/iforest/02-encoding-bigbase-iforest.py --dataset bigbase --version v1 --model iforest --config analysis/config/encoding-bigbase-iforest-v1.json

# Stage 3: Isolation Forest training with PCA
python analysis/iforest/03-training-iforest.py --dataset bigbase --version v1 --model iforest --sample-size 100000 --n-components 50

# Stage 4: Evaluation
python analysis/iforest/04-model-evaluation-iforest.py --dataset bigbase --version v1 --model iforest

# Stage 5: Feature space analysis
python analysis/iforest/05-feature-space-iforest.py --dataset bigbase --version v1 --model iforest
```

### GRU-SAE (GRU Autoencoder) Pipeline
```bash
# Stage 1: Data aggregation
python analysis/gru-sae/01-aggregation.py --model gru-sae --dataset unraveled --version v1 --schema analysis/config/schema-unraveled-gru-sae-v1.json

# Stage 2: GRU-specific sequence encoding
python analysis/gru-sae/02-encoding-gru-sae.py --dataset unraveled --version v1 --model gru-sae --encoding_config analysis/config/encoding-unraveled-gru-sae-v1.json --seq-len 10 --sample_size 20000

# Stage 3: GRU-SAE Training
python analysis/gru-sae/03-training-gru-sae.py --dataset unraveled --version v1 --model gru-sae --batch-size 16 --epochs 10

# Stage 4: GRU-SAE Evaluation  
python analysis/gru-sae/04-model-evaluation-gru-sae.py --dataset unraveled --version v1 --model gru-sae

# Stage 5: GRU-specific visualization
python analysis/gru-sae/05-result-vis-gru-sae.py --dataset unraveled --version v1 --model gru-sae
```

### Dataset Processing with Subdirectories
```bash
# For datasets with subdirectories (like unraveled network flows)
python analysis/sae/01-aggregation.py --model sae --dataset unraveled --version v1 --subdir network-flows --schema analysis/config/schema-unraveled-sae-v1.json
```

### Comparative Analysis
```bash
# Cross-model performance comparison
python analysis/comparative-evaluation/comparative_analysis.py

# Dataset quality analysis
python analysis/comparative-evaluation/dataset_quality_analyzer.py

# ROC curve comparisons
python analysis/comparative-evaluation/roc_comparison_generator.py
```

## Key Architecture Patterns

- **Model-specific directories**: Each anomaly detection approach (SAE, LSTM-SAE, Isolation Forest) has its own experiment directory
- **Version-based experimentation**: All artifacts use model-dataset-version naming (e.g., `sae-bigbase-v1`)
- **Modular encoding**: Different models use specialized encoding strategies:
  - SAE/Isolation Forest: 2D feature matrices with sparse storage
  - LSTM-SAE: 3D sequential data with temporal windowing
- **Memory optimization**: 
  - Sparse matrices for high-dimensional categorical features
  - Sampling strategies for large datasets (Isolation Forest, LSTM sequences)
  - PCA dimensionality reduction where appropriate
- **Benign-only training**: All anomaly detection models trained exclusively on normal samples
- **TensorFlow data generators**: Batch processing for neural networks without memory overflow
- **Explicit cleanup**: Memory management with `gc.collect()` and `K.clear_session()`

## Model-Specific Notes

### SAE (Stacked Autoencoder)
- Works on 2D feature matrices (samples × features)
- Uses dense layers with bottleneck architecture for dimensionality reduction
- Reconstruction error as anomaly score

### LSTM-SAE (LSTM Autoencoder) 
- Processes 3D sequences (samples × timesteps × features) 
- Incorporates temporal patterns in cybersecurity events
- Requires sequence building from chronologically sorted data
- Uses sampled sequences to manage memory with large datasets

### GRU-SAE (GRU Autoencoder)
- Processes 3D sequences (samples × timesteps × features) similar to LSTM-SAE
- Uses GRU cells for more efficient temporal modeling
- Incorporates temporal patterns with simpler architecture than LSTM
- Faster training compared to LSTM-SAE while maintaining sequence modeling capability

### Isolation Forest
- Applies PCA before training to reduce computational complexity
- Samples subset of training data (default: 100K samples, 50 PCA components)
- Isolation score as anomaly measure
- Faster training compared to deep learning approaches

## Dataset-Specific Notes

- **bigbase**: Windows event logs requiring TF-IDF for text columns (CommandLine, Image paths). Extended v2 includes additional text features.
- **unraveled**: Network flow data with categorical (IP/MAC addresses) and numeric (ports, bytes, packets) features. Includes temporal timestamps for LSTM sequencing.
- **dapt2020**: Pre-processed network flows with 85 standardized CICFlowMeter features
- All datasets use binary labeling with exact string matching for benign vs. attack classification




# Using Gemini CLI for Large Codebase Analysis

When analyzing large codebases or multiple files that might exceed context limits, use the Gemini CLI with its massive
context window. Use `gemini -p` to leverage Google Gemini's large context capacity.

## File and Directory Inclusion Syntax

Use the `@` syntax to include files and directories in your Gemini prompts. The paths should be relative to WHERE you run the
  gemini command:

### Examples:

**Single file analysis:**
```bash
gemini -p "@src/main.py Explain this file's purpose and structure"

Multiple files:
gemini -p "@package.json @src/index.js Analyze the dependencies used in the code"

Entire directory:
gemini -p "@src/ Summarize the architecture of this codebase"

Multiple directories:
gemini -p "@src/ @tests/ Analyze test coverage for the source code"

Current directory and subdirectories:
gemini -p "@./ Give me an overview of this entire project"

#
Or use --all_files flag:
gemini --all_files -p "Analyze the project structure and dependencies"

Implementation Verification Examples

Check if a feature is implemented:
gemini -p "@src/ @lib/ Has dark mode been implemented in this codebase? Show me the relevant files and functions"

Verify authentication implementation:
gemini -p "@src/ @middleware/ Is JWT authentication implemented? List all auth-related endpoints and middleware"

Check for specific patterns:
gemini -p "@src/ Are there any React hooks that handle WebSocket connections? List them with file paths"

Verify error handling:
gemini -p "@src/ @api/ Is proper error handling implemented for all API endpoints? Show examples of try-catch blocks"

Check for rate limiting:
gemini -p "@backend/ @middleware/ Is rate limiting implemented for the API? Show the implementation details"

Verify caching strategy:
gemini -p "@src/ @lib/ @services/ Is Redis caching implemented? List all cache-related functions and their usage"

Check for specific security measures:
gemini -p "@src/ @api/ Are SQL injection protections implemented? Show how user inputs are sanitized"

Verify test coverage for features:
gemini -p "@src/payment/ @tests/ Is the payment processing module fully tested? List all test cases"

When to Use Gemini CLI

Use gemini -p when:
- Analyzing entire codebases or large directories
- Comparing multiple large files
- Need to understand project-wide patterns or architecture
- Current context window is insufficient for the task
- Working with files totaling more than 100KB
- Verifying if specific features, patterns, or security measures are implemented
- Checking for the presence of certain coding patterns across the entire codebase

Important Notes

- Paths in @ syntax are relative to your current working directory when invoking gemini
- The CLI will include file contents directly in the context
- No need for --yolo flag for read-only analysis
- Gemini's context window can handle entire codebases that would overflow Claude's context
- When checking implementations, be specific about what you're looking for to get accurate results # Using Gemini CLI for Large Codebase Analysis


When analyzing large codebases or multiple files that might exceed context limits, use the Gemini CLI with its massive
context window. Use `gemini -p` to leverage Google Gemini's large context capacity.


## File and Directory Inclusion Syntax


Use the `@` syntax to include files and directories in your Gemini prompts. The paths should be relative to WHERE you run the
  gemini command:


### Examples:


**Single file analysis:**
```bash
gemini -p "@src/main.py Explain this file's purpose and structure"


Multiple files:
gemini -p "@package.json @src/index.js Analyze the dependencies used in the code"


Entire directory:
gemini -p "@src/ Summarize the architecture of this codebase"


Multiple directories:
gemini -p "@src/ @tests/ Analyze test coverage for the source code"


Current directory and subdirectories:
gemini -p "@./ Give me an overview of this entire project"
# Or use --all_files flag:
gemini --all_files -p "Analyze the project structure and dependencies"


Implementation Verification Examples


Check if a feature is implemented:
gemini -p "@src/ @lib/ Has dark mode been implemented in this codebase? Show me the relevant files and functions"


Verify authentication implementation:
gemini -p "@src/ @middleware/ Is JWT authentication implemented? List all auth-related endpoints and middleware"


Check for specific patterns:
gemini -p "@src/ Are there any React hooks that handle WebSocket connections? List them with file paths"


Verify error handling:
gemini -p "@src/ @api/ Is proper error handling implemented for all API endpoints? Show examples of try-catch blocks"


Check for rate limiting:
gemini -p "@backend/ @middleware/ Is rate limiting implemented for the API? Show the implementation details"


Verify caching strategy:
gemini -p "@src/ @lib/ @services/ Is Redis caching implemented? List all cache-related functions and their usage"


Check for specific security measures:
gemini -p "@src/ @api/ Are SQL injection protections implemented? Show how user inputs are sanitized"


Verify test coverage for features:
gemini -p "@src/payment/ @tests/ Is the payment processing module fully tested? List all test cases"


When to Use Gemini CLI


Use gemini -p when:
- Analyzing entire codebases or large directories
- Comparing multiple large files
- Need to understand project-wide patterns or architecture
- Current context window is insufficient for the task
- Working with files totaling more than 100KB
- Verifying if specific features, patterns, or security measures are implemented
- Checking for the presence of certain coding patterns across the entire codebase


Important Notes


- Paths in @ syntax are relative to your current working directory when invoking gemini
- The CLI will include file contents directly in the context
- No need for --yolo flag for read-only analysis
- Gemini's context window can handle entire codebases that would overflow Claude's context
- When checking implementations, be specific about what you're looking for to get accurate results