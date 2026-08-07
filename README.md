# esm2-vs-conservation-variant-benchmark
Benchmarking ESM-2 protein language model embeddings against MSA-based conservation scores for predicting pathogenic missense variants.

Abstract
Distinguishing pathogenic from benign missense variants remains a core challenge in clinical and computational genomics. While classical multiple-sequence-alignment (MSA)-based conservation scores have long served as a proxy for functional constraint, protein language models such as ESM-2 offer an alignment-free alternative learned directly from sequence data. Here, we benchmark ESM-2 embedding-based variant scores against MSA-derived conservation scores on a curated set of ClinVar-annotated missense variants across a panel of well-characterized human proteins. Using cross-validated logistic regression and standard classification metrics (AUROC, AUPRC), we quantify the relative and combined predictive power of each approach, and characterize cases where one method succeeds where the other fails. This work provides a reproducible, extensible pipeline for evaluating protein language models against established conservation-based baselines, offering practical guidance for their use in variant effect prediction.
Workflow
ClinVar pathogenic/benign missense variants are mapped to UniProt/RefSeq canonical sequences. In parallel, Pfam seed alignments or an MMseqs2 homolog search are used to build an MSA and compute a conservation score per residue, while ESM-2 embeddings are generated for wild-type versus mutant sequences to produce an embedding distance or masked-marginal score per variant. Both scores are merged into a single variant table, used to train and cross-validate a logistic regression classifier, and evaluated via AUROC/AUPRC comparison and error analysis of discordant cases. Results feed into the final figures and manuscript.
Step-by-step:
Data curation: Retrieve labeled missense variants from ClinVar; map to canonical sequences via UniProt/RefSeq.
Score generation: Compute ESM-2-based variant scores and MSA-based conservation scores in parallel.
Modeling and evaluation: Merge both scores per variant; train and evaluate logistic regression classifiers (individual and ensemble) with cross-validation.
Analysis and visualization: Compare performance metrics, plot ROC/PR curves, and inspect discordant predictions for biological insight.
Repository Structure
data/raw/ holds downloaded ClinVar, UniProt, and Pfam files (not committed, see Data Availability below). data/processed/ holds cleaned variant tables and alignments. scripts/ contains 01_fetch_variants.py, 02_conservation_scores.R, 03_esm2_scores.py, 04_model_eval.py, and 05_figures.py, run in that order. results/figures/ and results/tables/ hold outputs. environment.yml (or requirements.txt) and README.md sit at the repository root.

How to Reproduce

1. Environment setup
Clone the repository, then create and activate the environment:
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
conda env create -f environment.yml
conda activate variant-benchmark

2. Download data
Fetch the ClinVar variant summary:
python scripts/01_fetch_variants.py --output data/raw/clinvar_variants.tsv
Fetch UniProt canonical sequences for the target protein list:
python scripts/01_fetch_variants.py --sequences --output data/raw/sequences.fasta

3. Generate conservation scores
Requires the Bioconductor packages Biostrings and msa:
Rscript scripts/02_conservation_scores.R --fasta data/raw/sequences.fasta --output data/processed/conservation_scores.tsv

4. Generate ESM-2 scores
Uses fair-esm with the esm2_t33_650M_UR50D model; a GPU is recommended and the Colab free tier works fine:
python scripts/03_esm2_scores.py --variants data/raw/clinvar_variants.tsv --sequences data/raw/sequences.fasta --output data/processed/esm2_scores.tsv

5. Run evaluation and generate figures
python scripts/04_model_eval.py --conservation data/processed/conservation_scores.tsv --esm2 data/processed/esm2_scores.tsv --output results/tables/model_performance.tsv
python scripts/05_figures.py --input results/tables/model_performance.tsv --output results/figures/
Data Availability
Raw data are not committed to this repository due to size and licensing constraints; the download commands above regenerate them from public sources (ClinVar, UniProt, Pfam). Processed score tables used in the manuscript are provided in results/tables/.

Citation
If you use this pipeline, please cite the associated preprint: [bioRxiv link,  ]

License
MIT (or your preferred license)