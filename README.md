# esm2-vs-conservation-variant-benchmark
Benchmarking ESM-2 protein language model embeddings against MSA-based conservation scores for predicting pathogenic missense variants.

Abstract
Distinguishing pathogenic from benign missense variants remains a core challenge in clinical and computational genomics. While classical multiple-sequence-alignment (MSA)-based conservation scores have long served as a proxy for functional constraint, protein language models such as ESM-2 offer an alignment-free alternative learned directly from sequence data. Here, we benchmark ESM-2 embedding-based variant scores against MSA-derived conservation scores on a curated set of ClinVar-annotated missense variants across a panel of well-characterized human proteins. Using cross-validated logistic regression and standard classification metrics (AUROC, AUPRC), we quantify the relative and combined predictive power of each approach, and characterize cases where one method succeeds where the other fails. This work provides a reproducible, extensible pipeline for evaluating protein language models against established conservation-based baselines, offering practical guidance for their use in variant effect prediction.

Workflow
flowchart TD
    A[ClinVar: pathogenic/benign missense variants] --> B[Map variants to UniProt/RefSeq canonical sequences]
    C[Pfam seed alignments / MMseqs2 homolog search] --> D[MSA conservation score per residue]
    B --> E[ESM-2 embeddings: WT vs mutant sequence]
    E --> F[Embedding distance / masked-marginal score per variant]
    D --> G[Merge scores into single variant table]
    F --> G
    G --> H[Logistic regression + cross-validation]
    H --> I[AUROC / AUPRC comparison]
    H --> J[Error analysis: discordant cases]
    I --> K[Figures + manuscript]
    J --> K

Steps:
Data curation — Retrieve labeled missense variants from ClinVar; map to canonical sequences via UniProt/RefSeq.
Score generation — Compute (a) ESM-2-based variant scores and (b) MSA-based conservation scores in parallel.
Modeling & evaluation — Merge both scores per variant; train/evaluate logistic regression classifiers (individual and ensemble) with cross-validation.
Analysis & visualization — Compare performance metrics, plot ROC/PR curves, and inspect discordant predictions for biological insight.

Repository Structure

├── data/
│   ├── raw/              # Downloaded ClinVar, UniProt, Pfam files (not committed — see below)
│   └── processed/        # Cleaned variant tables, alignments
├── scripts/
│   ├── 01_fetch_variants.py
│   ├── 02_conservation_scores.R
│   ├── 03_esm2_scores.py
│   ├── 04_model_eval.py
│   └── 05_figures.py
├── results/
│   ├── figures/
│   └── tables/
├── environment.yml       # or requirements.txt
└── README.md

How to Reproduce

1. Environment setup
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
conda env create -f environment.yml
conda activate variant-benchmark

2. Download data
# ClinVar variant summary
python scripts/01_fetch_variants.py --output data/raw/clinvar_variants.tsv

# UniProt canonical sequences for target protein list
python scripts/01_fetch_variants.py --sequences --output data/raw/sequences.fasta

3. Generate conservation scores
# Requires Bioconductor packages: Biostrings, msa
Rscript scripts/02_conservation_scores.R \
  --fasta data/raw/sequences.fasta \
  --output data/processed/conservation_scores.tsv

4.Generate ESM-2 scores
# Uses fair-esm (esm2_t33_650M_UR50D) — GPU recommended (Colab free tier works)
python scripts/03_esm2_scores.py \
  --variants data/raw/clinvar_variants.tsv \
  --sequences data/raw/sequences.fasta \
  --output data/processed/esm2_scores.tsv

5. Run evaluation and generate figures
python scripts/04_model_eval.py \
  --conservation data/processed/conservation_scores.tsv \
  --esm2 data/processed/esm2_scores.tsv \
  --output results/tables/model_performance.tsv

python scripts/05_figures.py --input results/tables/model_performance.tsv --output results/figures/

Data Availability
Raw data are not committed to this repository due to size/licensing; download scripts above regenerate them from public sources (ClinVar, UniProt, Pfam). Processed score tables used in the manuscript are provided in results/tables/.
Citation
If you use this pipeline, please cite the associated preprint: [bioRxiv link —         ]
License
MIT 