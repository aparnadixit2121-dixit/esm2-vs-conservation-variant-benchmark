Benchmarking ESM-2 Embeddings vs. Conservation-Based Scores for Missense Variant Effect Prediction

Benchmarking ESM-2 protein language model embeddings against MSA-based conservation scores for predicting pathogenic missense variants.

Abstract

Distinguishing pathogenic from benign missense variants remains a core challenge in clinical and computational genomics. While classical multiple-sequence-alignment (MSA)-based conservation scores have long served as a proxy for functional constraint, protein language models such as ESM-2 offer an alignment-free alternative learned directly from sequence data. Here, we benchmark ESM-2 embedding-based variant scores against MSA-derived conservation scores on 6,498 ClinVar-annotated missense variants across 17 well-characterized human proteins. ESM-2 substantially outperformed conservation alone (AUROC 0.915 vs. 0.723), and a combined model offered only marginal additional benefit (AUROC 0.919). Error analysis revealed that ESM-2's advantage was concentrated in structurally complex, disulfide-stabilized proteins (notably FBN1), while conservation retained an edge on small, deeply invariant, well-studied proteins (HBB, TP53, BRCA1, KRAS). This work provides a reproducible pipeline for evaluating protein language models against conservation-based baselines, along with practical, biologically grounded guidance for their use.

Key Results
Model	AUROC	AUPRC
Conservation only	0.723	0.815
ESM-2 only	0.915	0.954
Combined	0.919	0.959
Workflow

ClinVar pathogenic/benign missense variants are mapped to UniProt canonical sequences. In parallel, an MMseqs2 homolog search against Swiss-Prot followed by MAFFT alignment is used to compute a Shannon-entropy-based conservation score per residue, while ESM-2 (esm2_t33_650M_UR50D) computes a masked-marginal score per variant. Both scores are merged into a single variant table, used to train and cross-validate logistic regression classifiers, and evaluated via AUROC/AUPRC comparison and gene-level error analysis of discordant predictions. Results feed into the final figures and manuscript.

Step-by-step:

Data curation (WP1): Retrieve labeled missense variants from ClinVar; map to canonical sequences via UniProt.
Conservation scoring (WP2): Homolog search (MMseqs2) + alignment (MAFFT) + Shannon entropy per position.
ESM-2 scoring (WP3): Masked-marginal scoring using ESM-2, with a windowed approach for proteins exceeding the model's practical sequence length.
Modeling and evaluation (WP4): Merge both scores per variant; train and cross-validate logistic regression classifiers (individual and combined).
Error analysis (WP5): Identify and characterize discordant predictions, including gene-level patterns explaining where each method succeeds or fails.
Repository Structure

All notebooks, data, and results are stored at the repository root for simplicity:

01_data_curation.ipynb
02_conservation_scoring.ipynb
03_esm2_scoring.ipynb
04_modeling_comparison.ipynb
05_error_analysis.ipynb
data_processed/          Cleaned variant tables, conservation and ESM-2 scores
results_figures/          roc_curves.png, pr_curves.png, score_correlation.png
results_tables/           model_performance.tsv, gene_level_breakdown.tsv,
                          top_discordant_cases.tsv, top_conservation_wins_cases.tsv,
                          error_analysis_summary.tsv
manuscript_draft.md
requirements.txt
README.md

Raw downloads (ClinVar variant summary, Swiss-Prot FASTA, MMseqs2 database) are not committed due to size; see Data Availability below.

How to Reproduce
1. Environment setup

Clone the repository and install dependencies:

git clone https://github.com/aparnadixit2121-dixit/esm2-vs-conservation-variant-benchmark.git cd esm2-vs-conservation-variant-benchmark pip install -r requirements.txt

2. Run the notebooks in order

Open each notebook in Google Colab or Jupyter and run top to bottom:

01_data_curation.ipynb 02_conservation_scoring.ipynb 03_esm2_scoring.ipynb 04_modeling_comparison.ipynb 05_error_analysis.ipynb

Notebook 03 requires a GPU runtime (Colab's free T4 tier is sufficient). Each notebook reads its inputs from and writes its outputs to a shared data directory (Google Drive if run in Colab), so they must be run in order the first time through.

Data Availability

Raw data (ClinVar variant summary, Swiss-Prot FASTA, MMseqs2 database) are not committed to this repository due to size and licensing constraints; Notebooks 01 and 02 regenerate them from public sources (ClinVar, UniProt). Processed score tables and results used in the manuscript are provided in data_processed/ and results_tables/.

Citation

If you use this pipeline, please cite the associated preprint: [bioRxiv link, add once posted]

License

MIT


