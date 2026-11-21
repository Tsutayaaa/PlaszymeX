# **PlaszymeGNN**

---
[![WebApp](https://img.shields.io/badge/WebApp-Online-brightgreen?)](http://plaszyme.org/plaszyme)
[![iGEM](https://img.shields.io/badge/iGEM-XJTLU--AI--China-blue?logo=)](https://teams.igem.org/5580)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![GitHub Repo](https://img.shields.io/badge/GitHub-PlaszymeGNN-black?logo=github)](https://github.com/Tsutayaaa/Plaszyme)
[![iGEM GitLab](https://img.shields.io/badge/GitLab-XJTLU--AI--China-orange?logo=gitlab)](https://gitlab.igem.org/2025/software-tools/xjtlu-ai-china)

<!-- Temporary celebration: iGEM 2026 -->
> 🎉 **Congratulations to XJTLU-China — iGEM 2026 Best Model**
>
> The **XJTLU-AI-China / Plaszyme team** warmly congratulates **XJTLU-China** on receiving the **Best Model award at iGEM 2026**. We are delighted to celebrate this achievement with our fellow iGEM team.
>
> Their **dogmaOS** project explores how synonymous codon design and host tRNA-pool engineering can shape translation rhythm to support protein folding and improve soluble recombinant protein expression in *Escherichia coli*. Its modeling framework brings together protein structure, evolutionary evidence and ribosome-traffic analysis. The accompanying **catmaOS** software provides a local browser workspace for translation-rhythm analysis and comparison of synonymous coding-sequence choices.
>
> We commend the team's work connecting computational modeling with experimental design and wish them continued success.
>
> [![Project Wiki](https://img.shields.io/badge/iGEM_2026-Project_Wiki-2563EB?style=flat-square)](https://2026.igem.wiki/xjtlu-china/) [![Modeling Framework](https://img.shields.io/badge/dogmaOS-Modeling_Framework-7C3AED?style=flat-square)](https://2026.igem.wiki/xjtlu-china/model) [![catmaOS Source](https://img.shields.io/badge/catmaOS-Source_Code-FC6D26?style=flat-square&logo=gitlab&logoColor=white)](https://gitlab.igem.org/2026/software/xjtlu-china/software)
<!-- End temporary celebration -->

## **Overview**

---

**PlaszymeGNN** is a deep learning framework designed to predict **enzyme–plastic interactions**, supporting the discovery of novel plastic-degrading enzymes within the context of synthetic biology.  

**PlaszymeGNN** adopts a **dual-tower architecture** that accepts **arbitrary inputs of enzymes and polymers**.  
This enables predictions on **previously unseen plastic molecules**, providing broader generalization and novel discovery.  

<p align="center">
  <img src="./doc/Model Backbone Architecture.png" alt="PlaszymeGNN dual-tower architecture" width="75%">
</p>
<p align="center"><em>Figure: The dual-tower architecture of PlaszymeGNN (Protein tower + Polymer tower + Interaction head).</em></p>

- **Protein tower**: GNN / GVP backbones, with optional **ESM embeddings** for enriched sequence and structure representation.  
- **Polymer tower**: Graph-based representation with **polymer-optimized descriptors**.  
  We innovatively normalize traditional descriptors to better reflect polymer properties and to generalize beyond known categories.  
- **Interaction head**: Multiple fusion modules (cosine similarity, bilinear layers etc) to capture enzyme–polymer binding patterns.  



### **Applications**

- **Enzyme discovery for plastic degradation**  
  Accelerating the identification of novel enzymes for plastic degradation pathways.  

- **Synthetic biology pathway design**  
  Leveraging arbitrary enzyme–polymer predictions to guide metabolic pathway engineering in synthetic biology.  

- **Novel polymer screening**  
  Predicting enzyme interactions with **previously unseen plastic molecules**, enabling exploration of new biodegradable materials.  

- **Experimental prioritization**  
  Providing ranked recommendations of enzyme–polymer pairs to reduce experimental cost and time.  

- **Understanding recognition mechanisms**  
  Revealing structural principles of enzyme–plastic recognition to inform rational design and directed evolution.  

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Prediction](#prediction)
- [Training](#training)
- [Project Structure](#project-structure)
- [Acknowledgements](#acknowledgements)
- [License](#license)

## **Installation**

---

### 1.Clone Repository

```bash
git clone <https://github.com/Tsutayaaa/Plaszyme.git>
cd Plaszyme
```

### 2.Create Conda Environment
We recommend using **[Conda](https://docs.conda.io/en/latest/)** to manage environments.

The general dependencies are provided in [`environment.yml`](./environment.yml). 

```bash
conda env create -f environment.yml
conda activate plaszyme_gnn
```

### 3.Install PyTorch
<p>
  <a href="https://pytorch.org/">
    <img src="https://raw.githubusercontent.com/pyg-team/pyg_sphinx_theme/master/pyg_sphinx_theme/static/img/pytorch_logo.svg" 
         height="15" style="position: relative; top: 2px;"> PyTorch
  </a> is the deep learning backbone of this project.
</p>

Install the correct version of PyTorch for your system.
Refer to the [PyTorch official installation guide](https://pytorch.org/get-started/locally/).

Example (CUDA 12.8):
```bash
pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu128
```
CPU-only (or Mac):
```bash
pip3 install torch torchvision
```

### 4.Install **PyG** *(PyTorch Geometric)*
<p align="left">
  <img src="https://raw.githubusercontent.com/pyg-team/pyg_sphinx_theme/master/pyg_sphinx_theme/static/img/pyg_logo.svg" alt="PyG logo" height="15" style="vertical-align: middle; margin-right: 2px;">
  <a href="https://pytorch-geometric.readthedocs.io/en/latest/"><b>PyG</b> (<i>PyTorch Geometric</i>)</a> is the core library we use to build graph-based protein and polymer representations.
</p>

Install following the [PyG installation guide](https://pytorch-geometric.readthedocs.io/en/latest/install/installation.html).

### 5.Verify Installation
Check that PyTorch and PyG are installed correctly:
```bash
python -c "import torch, torch_geometric; print(torch.__version__, torch_geometric.__version__)"
```
Expected output:
```
2.7.1 2.6.1
```

### 6. Install Project (Editable Mode)

Finally, install **PlaszymeGNN** itself so that the `src/` modules  
can be imported properly in training and prediction scripts.

```bash
pip install -e .
```
This command uses the `pyproject.toml` configuration to register
the plaszyme package in editable mode.

### 7. Verify Import
After installation, check that the package can be imported successfully:
```bash
python -c "import plaszyme; print('PlaszymeGNN package loaded successfully')"
```
If no error is shown, you are ready to run the training and prediction scripts.

## **Prediction**

---

PlaszymeGNN provides a list-wise scoring and analysis pipeline for **enzyme–polymer** interactions via `scripts/predict_listwise.py`. It supports confidence metrics, rank-based outputs, and batch evaluation.

### Configure

Edit the constants at the top of `scripts/predict_listwise.py`:
```python
# --- Model & data paths ---
MODEL_PATH = "../weights/gnn_bilinear/best_bilinear.pt"  # <- REQUIRED: your trained .pt weights
PT_OUT_ROOT = "./data/processed/graphs_pt"               # cached graphs (.pt)
SDF_ROOT = "./data/plastics_sdf/10_mers"                 # plastic library (.sdf/.mol)
PDB_ROOT = os.getcwd()                                    # graph builder root (keep default)

# --- Prediction ---
OUTPUT_DIR = "../prediction_results"
DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
TOP_K = 10
INCLUDE_CONFIDENCE = True

# --- Plastics (two modes) ---
USE_ALL_SDF_PLASTICS = True   # when PLASTIC_INPUT=None, load all from SDF_ROOT
DEFAULT_PLASTICS = [ ... ]    # fallback when SDF_ROOT is empty
PLASTIC_INPUT = None          # provide explicit .sdf/.mol to override the library (see below)

# --- Proteins (PDB) ---
# PDB_INPUT accepts: single file / list of files / a directory
PDB_INPUT = "/path/to/one.pdb"  # or ["a.pdb", "b.pdb"] or "/path/to/pdb_dir"
```
About `MODEL_PATH` (your `.pt` weights)
- Point `MODEL_PATH` to the exact `.pt` checkpoint you want to use (e.g., `../weights/gnn_bilinear/best_bilinear.pt`).
- The backbone (GNN/GVP/Seq) and interaction head are inferred from the checkpoint’s internal cfg; you don’t need to set them separately in the script.

### Input
#### A) Protein (PDB)

Supports a single file, a list of files, or a directory (the script collects all `.pdb` files).

Examples:
```python
PDB_INPUT = "./data/sample_pdb/X0001.pdb"
PDB_INPUT = ["./data/sample_pdb/X0001.pdb", "./data/sample_pdb/X0002.pdb"]
PDB_INPUT = "./data/sample_pdb/"   # directory
```

Graphs are built via `GNNProteinGraphBuilder`/`GVPProteinGraphBuilder` and cached to `PT_OUT_ROOT`.

Notes:
- We **recommend** using **AlphaFold** or **ColabFold** predicted PDBs that contain:
  - Only **residue-level atomic coordinates** (no crystallographic water, ligands, or alternate conformations).
  - A **single chain** (default reader will prioritize chain **A** if multiple chains exist).
  - A **single model** (multi-model ensembles are not supported).
- Overly complex PDBs (e.g., multi-chain complexes, additional heteroatoms) may cause parsing errors or unexpected graph construction issues.

#### B) Polymer (SDF/MOL)

Two modes are supported—explicit input takes precedence:

1. Library mode (default)
- If `PLASTIC_INPUT = None`, all `.sdf`/`.mol` files under `SDF_ROOT` are used.
- `SDF_ROOT` is empty, the script falls back to `DEFAULT_PLASTICS` (names are resolved as SDF_ROOT/{name}.sdf/.mol if present).
2. licit files (override library)
- `PLASTIC_INPUT` to a file, a list (files and/or folders), or a folder. The script collects all `.sdf`/`.mol` it finds and ignores `SDF_ROOT`/`DEFAULT_PLASTICS`.

```python
PLASTIC_INPUT = "/data/polymers/PET.sdf"                # single file
PLASTIC_INPUT = ["/data/polymers/PET.sdf", "/data/custom_dir"]  # mix of files/folders
PLASTIC_INPUT = "/data/polymers_dir"                    # folder only
```

Notes:
- The model natively supports **`.sdf`** and **`.mol`** formats.
- Structural complexity is **not strictly required** — minimal coordinate files are sufficient for descriptor extraction.
- **Advanced option**: By making a light modification to the featurizer code, you can also provide **SMILES strings** directly as polymer inputs instead of `.sdf/.mol`.  
  (This is disabled by default to maintain consistency, but can be adapted if you only have SMILES libraries.)

### Typical Scenarios

1. Single protein vs entire library
```python
PDB_INPUT = "/data/pdbs/X0001.pdb"
PLASTIC_INPUT = None
```
2. Many proteins vs custom SDF folder
```python
PDB_INPUT = "/data/pdbs/"
PLASTIC_INPUT = "/data/my_polymers_sdf/"
```
3. Single protein vs single SDF
```python
PDB_INPUT = "/data/pdbs/X0001.pdb"
PLASTIC_INPUT = "/data/custom/PET.sdf"
TOP_K = 1
```

### Outputs

For each protein, the script writes a CSV to `OUTPUT_DIR`:
`{PDB_BASENAME}_predictions.csv`, sorted by interaction_score (top TOP_K rows).

If `INCLUDE_CONFIDENCE=True`, the following columns are produced:

| Column               | Meaning                                                                 |
|-----------------------|-------------------------------------------------------------------------|
| **rank**             | Rank by `interaction_score` (descending)                                |
| **plastic_name**     | Polymer name or explicit file stem                                      |
| **interaction_score**| Interaction score from the interaction head                            |
| **confidence_score** | Softmax over scores in the batch *(T=0.1)*                              |
| **relative_strength**| Min–max normalized score                                                |
| **score_percentile** | Percentile within the batch score distribution                         |
| **embedding_similarity** | Cosine similarity in the projected feature space                     |
| **prediction_category**  | Rule-based label *(High / Medium / Low / No Significant Interaction)* |

Single-plastic special case: if only one plastic is scored, the category reduces to a simple threshold:
interaction_score > 0 → High Interaction; otherwise No Significant Interaction.

### Tuning
```python
DEVICE = "cuda" / "cpu"     # auto-detected by default; can force it
TOP_K = 10                  # export top-K rows only
INCLUDE_CONFIDENCE = True   # include confidence & analysis columns
VERBOSE = True              # logging verbosity
```
The active backbone and interaction head are defined by the loaded checkpoint’s cfg (inside `MODEL_PATH`).

### Troubleshooting
- Model file not found: check the `MODEL_PATH` absolute/relative path and filename.
- No PDB files found: verify `PDB_INPUT` type/path and `.pdb` suffix.
- No valid plastic features extracted: ensure valid `.sdf`/`.mol` (or correct `SDF_ROOT`).
- RDKit errors: prefer installing rdkit via conda-forge for best compatibility.
- Graph build failed: the script will try to load cached graphs from `PT_OUT_ROOT`.
- Unxpected category with a single plastic: the single-item case uses the strict threshold (score > 0).

## **Training**

---

### Quick Start
```bash
# 1) Activate env and install the project (editable)
conda activate plaszyme_gnn
pip install -e .

# 2) Run with defaults (paths in the script header)
python scripts/train_listwise.py
```
By default, the script uses the hyperparameters and paths defined in TrainConfig inside train_listwise.py.
Customize via a JSON config to avoid editing code:
```bash
python scripts/train_listwise.py --config configs/train_listwise.json
```
Example `configs/train_listwise.json` (edit paths to your data):
```json
{
  "device": "cuda",
  "enz_backbone": "GNN",
  "pdb_root": "./data/sample_pdb",
  "pt_out_root": "./data/processed/graphs_pt",
  "sdf_root": "./data/plastics_sdf/10_mers",
  "train_csv": "./data/matrix/plastics_onehot_trainset.csv",
  "test_csv": "./data/matrix/plastics_onehot_testset.csv",
  "out_dir": "./train_results/gnn_bilinear",
  "emb_dim_enz": 128,
  "emb_dim_pl": 128,
  "proj_dim": 128,
  "batch_size": 10,
  "lr": 3e-4,
  "weight_decay": 1e-4,
  "epochs": 100,
  "max_list_len": 10,
  "temp": 0.2,
  "alpha": 0.4,
  "interaction": "gated",
  "bilinear_rank": 64,
  "lambda_w_reg": 1e-4,
  "enable_list_mitigation": false,
  "seed": 42
}
```
### Data Preparation
#### Enzymes (PDB structures)

- We recommend using **AlphaFold/ColabFold predictions**:  
  - **Single chain**, **single model**, **residue coordinates only**.  
  - If a PDB contains multiple chains/models, the builder defaults to **chain A**.  
- Example folder: `./data/sample_pdb`

#### Plastics

- Provided as **SDF/MOL oligomers** (e.g., 10-mers) in:  
  `./data/plastics_sdf/10_mers`  
- Internal featurizer computes **polymer-optimized, normalized descriptors**.  
- Advanced users: accept **SMILES input** with minor code changes.

#### Training Matrix (CSV)

- Format: rows = enzymes, columns = plastics, entries = {0,1} indicating known degradation.
- Example:

| enzyme_id     | PET | PCL | PLA | PBAT | ... |
|---------------|-----|-----|-----|------|------|
| X0004         | 1   | 0   | 0   | 1    |...|
| X0006         | 0   | 1   | 0   | 0    |...|
| X0009         | 0   | 0   | 1   | 0    |...|
| X0010         | 0   | 0   | 1   | 0    |...|
| ...           | ... | ... | ... | ...  |...|
| X0020         | 1   | 0   | 0   | 0    |...|
| X0021         | 0   | 1   | 0   | 0    |...|
| X0024         | 0   | 0   | 1   | 0    |...|

Here:
- `enzyme_id` = PDB filename
- Each plastic column = polymer class
- Value `1` = enzyme reported to degrade this polymer

#### Data Source

All example datasets can be constructed or extended from **PlaszymeDB**,  
a curated database of **plastic-degrading enzymes, polymers, and their reported interactions**.
> **PlaszymeDB Sources**  
[![GitHub](https://img.shields.io/badge/PlaszymeDB-GitHub-blue?logo=github)](https://github.com/Tsutayaaa/PlaszymeDB)
[![WebApp](https://img.shields.io/badge/PlaszymeDB-WebApp-brightgreen?logo=firefox)](http://plaszyme.org/plaszymeDB)
> 
> GitHub: [https://github.com/Tsutayaaa/PlaszymeDB](https://github.com/Tsutayaaa/PlaszymeDB)  
> WebApp: [http://plaszyme.org/plaszymeDB](http://plaszyme.org/plaszymeDB)  

### Architecture Options

PlaszymeGNN adopts a **dual-tower** architecture:

- **Protein tower (enzyme encoder)**
  - `"GNN"` — graph convolutional backbone (default)
  - `"GVP"` — geometric vector perceptron (vector/scalar features)
  - `"MLP"` — sequence pooling + MLP baseline  
  (All support optional ESM embeddings during graph building.)

- **Polymer tower (plastic encoder)**
  - `PolymerTower` that consumes **polymer-optimized, normalized descriptors** (from SDF/MOL; lengths padded).

- **TwinProjector**
  - `TwinProjector` maps both towers into the same latent space (linear, Xavier init).

- **Interaction head**
  - Fuses enzyme & plastic embeddings into a scalar score.

Minimal JSON to choose the protein tower:

    { "enz_backbone": "GNN" }

### Interaction Heads (fusion)

Select via `interaction`:

| Name                  | Key                     | Mechanism                        | Notes                              |
|----------------------|-------------------------|----------------------------------|------------------------------------|
| Cosine               | `"cos"`                  | cosine(z_e, z_p)                 | Simple & stable baseline           |
| Bilinear             | `"bilinear"`            | eᵀ **W** p                       | Strong but may overfit; add reg    |
| Factorized Bilinear  | `"factorized_bilinear"` | eᵀ (**UᵀV**) p (low-rank)        | Parameter-efficient; add ortho     |
| Hadamard             | `"hadamard_mlp"`        | (e ⊙ p) ↦ linear                 | Lightweight, fast                  |
| Gated (default)      | `"gated"`               | learnable gating on e ⊙ p        | Good accuracy-speed tradeoff       |

Example snippet:

    {
      "interaction": "gated",
      "bilinear_rank": 64,
      "lambda_w_reg": 1e-4,
      "ortho_reg": 0.0
    }

### Listwise Training Objective

- **Per-enzyme list** of plastics trained with **listwise InfoNCE**.
- Temperature `temp` controls sharpness.
- **Diversity** on positive plastics reduces mode collapse.
- **Center/variance** regularization stabilizes scales.

Typical settings:

    {
      "temp": 0.2,
      "alpha": 0.4,
      "plastic_diversify": true,
      "lambda_diversify": 0.05,
      "var_target": 1.0,
      "lambda_center": 1e-3
    }

### List-Mitigation (sampling optimization)

Imbalance in long lists is mitigated by:
- Keeping **all positives**.
- Mixing **hard negatives** (highest scores) with random negatives.
- Normalizing by sublist length.

Enable & tune:

    {
      "enable_list_mitigation": true,
      "max_list_len_train": 10,
      "neg_per_item": 32,
      "hard_neg_cand": 32,
      "hard_neg_ratio": 0.5
    }

###  Polymer Tower Pretraining (optional)
In addition to joint training with enzymes, the **polymer tower** can be **pretrained independently** using a **plastic co-occurrence matrix** (e.g., from biodegradation assays or curated databases).  

- **Why it works:**  
  Plastics that are frequently degraded together by similar enzymes tend to share **chemical substructures** and **functional motifs**.  
  By learning to **approximate this co-occurrence similarity**, the polymer tower acquires a **chemically meaningful embedding space** even before being paired with enzymes.  

- **Benefits:**  
  - Provides a stronger initialization for downstream enzyme–plastic prediction.  
  - Improves generalization to **rare or unseen polymers**.  
  - Reduces overfitting when training data is sparse.  

Optionally pretrain the polymer tower using a **plastic co-occurrence matrix**:

    {
      "use_plastic_pretrain": true,
      "co_matrix_csv": "./data/matrix/plastic_co_matrix.csv",
      "pretrain_epochs": 10,
      "pretrain_loss_mode": "contrastive"   // or "mse"
    }

The script exports a `plastic_pretrained.pt` in `out_dir` after pretraining.

### Outputs

- **Logs**: `train.log`, `run_config.txt`, `run_config.json`
- **Checkpoints**: `best_<interaction>.pt`, `last_<interaction>.pt`
- **Curves**: `loss.png`, `hit.png`, `score.png`
- **Evaluation**: `test_metrics.csv` and optional `test_score_matrix.csv`

The script will automatically evaluate on `test_csv` using the **best** checkpoint.

### Practical Tips

- Start with **`GNN + gated`**; then try **`factorized_bilinear`** for sharper ranking with fewer params.
- Clean PDBs reduce graph-build errors (avoid multi-model, altloc clutter).
- Fix seeds for reproducibility.
- Watch `emb_diag.csv` (generated internally) for embedding collapse or scale drift.

## Project Structure

---

The repository follows a modular design to separate **core library code**, **training/evaluation scripts**, and **documentation assets**.

```bash
├── LICENSE
├── README.md
├── doc/
├── environment.yml              # Conda environment specification
├── pyproject.toml               # Python package configuration (PEP 621)
├── data/
│   ├── plastics_sdf/               # Polymer structure files
│   │   ├── 10_mers/                # Polymer fragments (10-mer SDF format)
│   │   └── 3_mers/                 # Smaller polymer fragments (3-mer MOL format)
│   ├── processed/                  # Pre-processed graph representations
│   │   └── graphs_pt/              # PyTorch Geometric graph tensors
│   └── sample_pdb/                 # Example enzyme PDB structures
├── scripts/                     # Training and prediction entry points
│   ├── evaluate_tools/
│   │   └── evaluate_testset.py  # Evaluation utilities for test sets
│   ├── predict_listwise.py      # Prediction script for enzyme–plastic interactions
│   └── train_listwise.py        # Main training script
├── src/                         # Source code (installed as `plaszyme`)
│   └── plaszyme/
│       ├── builders/            # Protein graph construction modules
│       │   ├── base_builder.py        # Base builder class (common logic)
│       │   ├── gnn_builder.py         # Graph builder for GNN-based models
│       │   ├── gvp_builder.py         # Graph builder for GVP (vector-scalar graphs)
│       │   └── sequence_embedder.py   # Sequence embedding (ESM / one-hot / custom)
│       │
│       ├── heads/               # Residue-level supervision heads
│       │   ├── residue_activity_head.py  # Predict residue activity/intensity
│       │   └── residue_role_head.py      # Predict residue roles (interaction/reactant/spectator)
│       │
│       ├── models/              # Backbone networks and interaction modules
│       │   ├── gnn/                  # Graph Neural Network backbones
│       │   │   └── backbone.py
│       │   ├── gvp/                  # Geometric Vector Perceptron backbones
│       │   │   ├── backbone.py
│       │   │   └── gvp_local/        # Atom-level GVP extensions
│       │   │       ├── atom3d.py
│       │   │       ├── data.py
│       │   │       └── models.py
│       │   ├── seq_mlp/              # Baseline sequence-only encoder
│       │   │   └── backbone.py
│       │   ├── plastic_backbone.py   # Polymer Tower (plastic feature encoder)
│       │   └── interaction_head.py   # Fusion layers (cosine, bilinear, gated, etc.)
│       │
│       ├── plastic/             # Polymer feature extraction
│       │   ├── descriptors_rdkit.py  # RDKit-based descriptor featurizer
│       │   └── rdkit_features.yaml   # Descriptor configuration file
│       │
│       ├── readers/             # Structure parsing utilities
│       │   └── pdb_reader.py         # Optimized for AlphaFold/ColabFold single-chain PDBs
│       │
│       └── viz_graph.py         # Protein–polymer graph visualization
│
└── weights/                     # Saved weights (multiple interaction heads)
```
### Module Breakdown

#### `builders/`
Protein graph construction and embeddings
- `base_builder.py` → Base class for graph builders  
- `gnn_builder.py` → Builds residue-level graphs for GNN  
- `gvp_builder.py` → Builds vector–scalar graphs for GVP  
- `sequence_embedder.py` → Sequence-level embeddings (ESM / one-hot)

#### `heads/`
Residue-level supervision modules
- `residue_activity_head.py` → Predict activity scores  
- `residue_role_head.py` → Predict functional roles (interaction / spectator)

#### `models/`
Core neural architectures
- `gnn/backbone.py` → Graph Neural Network backbone  
- `gvp/backbone.py` → Geometric Vector Perceptron backbone  
- `gvp_local/` → Atom-level extensions for GVP  
- `seq_mlp/backbone.py` → Baseline sequence MLP  
- `plastic_backbone.py` → Polymer Tower (plastic descriptors → embeddings)  
- `interaction_head.py` → Flexible scoring heads (cosine, bilinear, gated, etc.)

#### `plastic/`
Polymer-specific feature extraction
- `descriptors_rdkit.py` → RDKit-based descriptor featurizer  
- `rdkit_features.yaml` → Descriptor config file

#### `readers/`
Input parsing utilities
- `pdb_reader.py` → Parses PDB files (optimized for AlphaFold / ColabFold single-chain structures)

#### `viz_graph.py`
Visualization tool for protein–polymer interaction graphs.

## Acknowledgements

---

<p align="center">
  <img src="./doc/xjtlu-ai-china_logo_light.svg" 
       alt="XJTLU-AI-China Logo" 
       width="150" 
       style="vertical-align: middle; margin-right: 30px; filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.3));">
  <img src="./doc/iGEM_logo_light.jpg" 
       alt="iGEM Logo" 
       width="100" 
       style="vertical-align: middle; filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.3));">
</p>

This project is part of the **[iGEM 2025 Competition](https://igem.org/)**, developed by the **[XJTLU-AI-China](https://teams.igem.org/5580)** team.

This project was developed in the context of the **iGEM 2025 competition**.

- **XJTLU-AI-China iGEM 2025 Team**: for initiating and developing the PlaszymeGNN project.  
- **School of Science, Xi’an Jiaotong-Liverpool University (XJTLU)**: for institutional support.  
- **Prof. [Chun Chan](https://scholar.xjtlu.edu.cn/en/persons/ChunChan)** (Kevin) (School of Science, XJTLU), our **Principal Investigator (PI)**: for providing invaluable guidance and mentorship throughout the project.
- **[GVP](https://github.com/drorlab/gvp-pytorch)**: for providing the Geometric Vector Perceptron backbone implementation.  
- **[ESM](https://github.com/facebookresearch/esm)**: for enabling powerful protein sequence and structure embeddings.  
- We also gratefully acknowledge the **open-source community**, whose tools and resources have made this work possible.  

## License

---

This project is licensed under the [MIT License](./LICENSE).  
You are free to use, modify, and distribute this project, provided that proper attribution is given.  