# esm2-vs-conservation-variant-benchmark
Benchmarking ESM-2 Embeddings vs. Conservation-Based Scores for Missense Variant Effect Prediction
Benchmarking ESM-2 protein language model embeddings against MSA-based conservation scores for predicting pathogenic missense variants

Abstract

Distinguishing pathogenic from benign missense variants remains a core challenge in clinical and computational genomics. While classical multiple-sequence-alignment (MSA)-based conservation scores have long served as a proxy for functional constraint, protein language models such as ESM-2 offer an alignment-free alternative learned directly from sequence data. Here, we benchmark ESM-2 embedding-based variant scores against MSA-derived conservation scores on 6,498 ClinVar-annotated missense variants across 17 well-characterized human proteins. ESM-2 substantially outperformed conservation alone (AUROC 0.915 vs. 0.723), and a combined model offered only marginal additional benefit (AUROC 0.919). Error analysis revealed that ESM-2's advantage was concentrated in structurally complex, disulfide-stabilized proteins (notably FBN1), while conservation retained an edge on small, deeply invariant, well-studied proteins (HBB, TP53, BRCA1, KRAS). This work provides a reproducible pipeline for evaluating protein language models against conservation-based baselines, along with practical, biologically grounded guidance for their use.


Key Results
Model	                     AUROC	   AUPRC
Conservation only	         0.723	   0.815
ESM-2 only	               0.915	   0.954
Combined	                 0.919	   0.959

Workflow

ClinVar pathogenic/benign missense variants are mapped to UniProt canonical sequences. In parallel, an MMseqs2 homolog search against Swiss-Prot followed by MAFFT alignment is used to compute a Shannon-entropy-based conservation score per residue, while ESM-2 (esm2_t33_650M_UR50D) computes a masked-marginal score per variant. Both scores are merged into a single variant table, used to train and cross-validate logistic regression classifiers, and evaluated via AUROC/AUPRC comparison and gene-level error analysis of discordant predictions. Results feed into the final figures and manuscript.

Step-by-step:

Data curation (WP1): Retrieve labeled missense variants from ClinVar; map to canonical sequences via UniProt.
Conservation scoring (WP2): Homolog search (MMseqs2) + alignment (MAFFT) + Shannon entropy per position.
ESM-2 scoring (WP3): Masked-marginal scoring using ESM-2, with a windowed approach for proteins exceeding the model's practical sequence length.
Modeling and evaluation (WP4): Merge both scores per variant; train and cross-validate logistic regression classifiers (individual and combined).
Error analysis (WP5): Identify and characterize discordant predictions, including gene-level patterns explaining where each method succeeds or fails.

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

Repository Structure
notebooks/
    01_data_curation.ipynb
    02_conservation_scoring.ipynb
    03_esm2_scoring.ipynb
    04_modeling_comparison.ipynb
    05_error_analysis.ipynb
data/
    processed/          Cleaned variant tables, conservation and ESM-2 scores
results/
    figures/             roc_curves.png, pr_curves.png, score_correlation.png
    tables/              model_performance.tsv, gene_level_breakdown.tsv,
                         top_discordant_cases.tsv, top_conservation_wins_cases.tsv,
                         error_analysis_summary.tsv
manuscript/
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

notebooks/ 01_data_curation.ipynb 
notebooks/ 02_conservation_scoring.ipynb 
notebooks/03_esm2_scoring.ipynb 
notebooks/04_modeling_comparison.ipynb 
notebooks/05_error_analysis.ipynb

Notebook 03 requires a GPU runtime (Colab's free T4 tier is sufficient). Each notebook reads its inputs from and writes its outputs to a shared data directory (Google Drive if run in Colab), so they must be run in order the first time through.

Data Availability

Raw data (ClinVar variant summary, Swiss-Prot FASTA, MMseqs2 database) are not committed to this repository due to size and licensing constraints; Notebooks 01 and 02 regenerate them from public sources (ClinVar, UniProt). Processed score tables and results used in the manuscript are provided in data/processed/ and results/.

Citation

If you use this pipeline, please cite the associated preprint: [bioRxiv link, add once posted]

License

MIT

















