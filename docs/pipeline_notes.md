# Pipeline notes: verification status and open items

This file tracks, per pipeline step, what has been verified against real executed output
and what still needs attention before submission. It is a working checklist, not a
methods section.

## Step-by-step verification status

| Step | What it does | Status |
|---|---|---|
| 2 | IEDB epitope query, QC, 932-pair pool | Verified — `step2_iedb_epitopes.csv`, `step2_iedb_query.log` are real query output |
| 3 | AF3 benchmark vs 5 crystal structures | Verified and corrected — see **Item 1** below |
| 4 | Full-panel AF3 submission, dedup, 35-pair shortlist | Verified — 932→584 unique→35-pair coverage-aware shortlist confirmed in notebook output and `step4_final_manifest.csv` |
| 5 | Groove geometry (PyMOL) | Verified — `Table_S2_groove_geometry_concordance.csv`, 35/35 pairs structure-ready |
| 6 | PRODIGY affinity | Verified — `Table_S3_prodigy_affinity.csv`, 35/35 rows `status=ok` |
| 7 | Structure-affinity consistency | Verified — `Table_S4_structure_affinity_correlations.csv`, non-significant in both classes as reported in the manuscript |
| 8 | Allele structural clustering | Verified, with a within-class control check added — see **Item 2** |
| 9 | Tissue expression (GTEx/HPA) | Verified — `Table_S6_*` tables real; see **Item 3** |

All 7 notebooks in `notebooks/` have been checked against the real executed output in
`results/tables/`, validated as well-formed, and corrected where the source disagreed
with that output.

## Items

### 1. Step 3 benchmark B5 corrected to PDB 1DLH (fixed)

`notebooks/MS3_step3_af3_benchmark.ipynb`'s `BENCHMARK_PAIRS` list previously defined B5
against PDB 1KLU. The real executed output (`Table_S0_af3_benchmark.csv`) shows B5 was
actually benchmarked against 1DLH (HLA-DR1/HA 306-318; TM-score 0.9902, RMSD 0.433 Å,
resolution 2.8 Å, 1994 deposition) — there is no 1KLU result anywhere in the data. The
notebook has been corrected so `BENCHMARK_PAIRS` reads 1DLH, with a note in the cell
documenting the correction, so the published code now matches the published numbers.

### 2. Step 8 clustering: within-class control added for the chain-count question

`Table_S5_allele_structural_aggregation.csv` shows zero overlap between class I and
class II allele means on every feature (BSA, contact count, polar contacts). Class II
complexes have two HLA chains contributing to the modelled interface (DRA+DRB1); class
I complexes have one (the heavy chain only). A larger structural interface for class II
is therefore expected from chain count alone, independent of any genuine allele-level
signal the clustering is meant to detect.

`notebooks/MS3_step8_allele_clustering.ipynb` now includes a Part E that re-runs Ward
clustering **separately within each class** (re-standardized within that class only) and
reports the largest within-class pairwise distance against the largest cross-class
distance. Run this cell and read its printed interpretation before finalising the
dendrogram figure/text: if within-class structure is small relative to the cross-class
split, present the combined dendrogram's I/II bifurcation as consistent with the
chain-count difference, not as evidence of allelic structural divergence beyond that —
a one-sentence hedge in the manuscript, not a result to drop.

### 3. Step 9: two genuine data-availability gaps, one modality pivot — check manuscript text matches

Three things to verify read correctly in the manuscript's Methods/Results, not just in
this repository:

- **No lymph node tissue.** Neither GTEx nor HPA includes a true lymph node organ
  sample. GTEx's nearest entry, "Cells - EBV-transformed lymphocytes," is a cultured
  cell line, not lymph node tissue, and was excluded rather than used as a substitute.
- **No upper-airway tissue.** GTEx's nearest entry, "Minor_Salivary_Gland," is exocrine
  gland tissue, not airway epithelium, and was likewise excluded rather than substituted.
- **HPA column is RNA-seq, not IHC.** HPA protein-level immunohistochemistry was
  confirmed absent from HPA's current public export (checked in both the bulk TSV and
  the per-gene XML endpoint). `Table_S6_hpa_hla_expression.csv` is HPA's independent
  tissue RNA-seq instead. This makes the GTEx/HPA comparison a same-modality,
  cross-pipeline replication, not a protein-vs-RNA cross-modality check. The notebook's
  own `PIVOT_NOTE` states this explicitly; the manuscript text should state it with the
  same explicitness, and should not describe this analysis as "spatial" in any sense
  (neither GTEx nor HPA tissue RNA-seq is spatially resolved).

**Action before submission:** re-read the manuscript's Step 9 Methods/Results paragraph
against these three points and adjust wording if any of them reads as a substitution
rather than an acknowledged gap, or implies IHC where RNA-seq was used.

## Notebooks

`notebooks/` holds the 7 analysis notebooks for Steps 3-9, each checked against its
corresponding table(s) in `results/tables/`:

| Notebook | Produces |
|---|---|
| `MS3_step3_af3_benchmark.ipynb` | `Table_S0_af3_benchmark.csv` |
| `MS3_step4_af3_full_panel.ipynb` | `step4_final_manifest.csv`, the 35-pair shortlist |
| `MS3_step5_groove_geometry.ipynb` | `Table_S2_groove_geometry_concordance.csv` |
| `MS3_step6_prodigy_affinity.ipynb` | `Table_S3_prodigy_affinity.csv` |
| `MS3_step7_structure_affinity_consistency.ipynb` | `Table_S4_structure_affinity_correlations.csv`, `Table_S4_structure_affinity_scatter_data.csv` |
| `MS3_step8_allele_clustering.ipynb` | `Table_S5_allele_structural_aggregation.csv`, `Table_S5_allele_pairwise_distances.csv`, `Figure_allele_structural_dendrogram.png` |
| `MS3_step9_tissue_expression.ipynb` | `Table_S6_gtex_hla_expression.csv`, `Table_S6_hpa_hla_expression.csv`, both tissue-expression figures |

Several of these notebooks have manual steps (AF3 server submission has no batch API;
NetMHCpan/NetMHCIIpan run locally, not in Colab) and Colab-specific cells (`google.colab`
file-upload widgets, Drive sync) that will not execute as-is outside Colab — these are
flagged inline in each notebook rather than stripped, since they reflect the real
execution environment the results were produced in.

Step 2 (IEDB query) and Step 10 (final integration) are referenced by these notebooks
but are not themselves included here.
