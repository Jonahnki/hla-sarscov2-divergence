# AlphaFold3-Predicted HLA–SARS-CoV-2 Peptide Complexes Reveal Class I/Class II Divergence in Peptide-Binding Groove Architecture

Reproducibility package for a structural immunogenomics study of HLA–SARS-CoV-2 peptide
complexes. AlphaFold3 structures are benchmarked against independent crystal structures,
peptide-binding groove geometry is characterised with PyMOL, binding affinity is
estimated with PRODIGY, and tissue-compartment HLA expression is profiled across GTEx
and the Human Protein Atlas.

---

## Overview

Sixteen HLA alleles spanning class I (HLA-A, HLA-B) and class II (HLA-DRB1) were paired
with SARS-CoV-2 epitopes drawn from the Immune Epitope Database (IEDB), structurally
predicted with AlphaFold3, and characterised across four independent analyses: structural
benchmarking, peptide-binding groove geometry, PRODIGY binding affinity, and allele-level
structural clustering. A fifth analysis profiles HLA gene expression across tissue
compartments using GTEx and the Human Protein Atlas, independent of structure prediction.

The central finding is a dominant, though not fully disjoint, structural bifurcation
between HLA class I and class II peptide-binding grooves on hierarchical clustering of
AF3-derived geometry features, alongside the expected absence of a strong structure–
affinity relationship within either class via groove geometry at this sample size (though
sequence-based NetMHCpan rank does correlate with affinity within class I), and
substantial, assay-modality-driven divergence between RNA-seq- and protein-level HLA
expression estimates across tissue compartments.

## Panel and pipeline

| | |
|---|---|
| **HLA alleles** | 16 (10 class I: HLA-A, HLA-B; 6 class II: HLA-DRB1) |
| **Allele–peptide pairs** | 35, shortlisted from 584 unique candidates (coverage-aware: ≥2 pairs/allele, up to 5 for HLA-A\*02:01) |
| **Epitope source** | IEDB (PostgREST API query, QC-filtered) |
| **Structure prediction** | AlphaFold3 (AF3 Server) |
| **Benchmark set** | 4 independent crystal structures (not drawn from the panel) |
| **Groove geometry** | PyMOL (open-source build), locus-aware chain treatment |
| **Binding affinity** | PRODIGY (BonvinLab), interface contact-based ΔG/Kd |
| **Tissue expression** | GTEx (API V2) and Human Protein Atlas (independent tissue RNA-seq) |

Pipeline stages are implemented as sequential Jupyter notebooks under [`notebooks/`](notebooks/),
each producing one or more of the supplementary tables in [`results/tables/`](results/tables/).
See [`docs/data_dictionary.md`](docs/data_dictionary.md) for column-level documentation of
every table and [`docs/pipeline_notes.md`](docs/pipeline_notes.md) for the verification
status of each stage.

## Key results

**Structural benchmark.** AF3-predicted structures for 4 independent HLA–peptide
complexes (not drawn from the analysis panel) were benchmarked against crystal structure
counterparts, with TM-score/RMSD confirmed for all four: B1 vs PDB 1OGA (TM-score 0.985,
RMSD 0.67 Å), B2 vs PDB 3GSO (TM-score 0.992, RMSD 0.54 Å), B4 vs PDB 3VCL (TM-score
0.963, RMSD 1.17 Å), and B5 vs PDB 1DLH (TM-score 0.990, RMSD 0.43 Å). B2 and B4 were
re-referenced during manuscript preparation from PDB IDs that turned out to be the wrong
crystal structures (1T1W and 1W0V are real structures, but of different allele/peptide
complexes than originally claimed) to the verified correct structures (3GSO and 3VCL
respectively), and their TM-score/RMSD recomputed against the corrected references using
two independent superposition methods (PyMOL CEAlign and TM-align) in close agreement
(see `docs/pipeline_notes.md` Item 1). A fifth planned benchmark (HLA-B\*07:02/RPRGEVRFL
vs PDB 3MGT, also a wrong reference) was dropped outright — no genuine crystal structure
of that peptide bound to any HLA allele could be found.

**Class I / class II structural bifurcation.** Hierarchical clustering (Ward linkage) of
z-scored allele-level groove geometry features — buried surface area, contact residue
count, polar contact count — produces a top-level split separating class I from class II
alleles, with the four largest pairwise structural distances in the full 16-allele
distance matrix all falling between a class I and a class II allele. This is not a
complete, disjoint separation: within-class pairwise distances (median 0.73 for class I,
1.17 for class II) are well below the cross-class median (3.26), but 14 of 60 cross-class
pairs (23%) fall below the largest within-class distance. Class II allele means (buried
surface area 2166–2317 Å²; 35.0–41.5 contact residues) exceed class I allele means
(1771–1855 Å²; 28.5–33.5 contact residues) on every feature's mean, consistent with class
II's two-chain (DRA + DRB1) contact interface versus class I's single-chain interface —
some portion of this separation is expected from chain count alone, and the within-class
distances confirm genuine allele-level structural variation exists within each class
independent of that confound.

**Structure–affinity consistency.** Spearman correlation between groove geometry
features and PRODIGY-predicted ΔG, computed separately within class I (n = 23 pairs) and
class II (n = 12 pairs), was non-significant across all six feature–class combinations
(ρ = −0.432 to 0.148, p = 0.16–0.91). This null result is reported as an honest,
power-limited finding rather than omitted, consistent with the small per-class sample
size. Sequence-based NetMHCpan/NetMHCIIpan eluted-ligand rank, by contrast, correlates
significantly with PRODIGY ΔG within class I (ρ = 0.586, p = 0.0033, n = 23) though not
within class II (ρ = 0.151, p = 0.64) or pooled across classes (ρ = 0.014, p = 0.94) —
sequence- and structure-based affinity predictions agree within class I even where
neither correlates with groove geometry.

**Tissue-compartment expression divergence.** GTEx (bulk RNA-seq) and HPA (independent
tissue RNA-seq) HLA gene expression estimates diverge by 0.69×–153.9× across shared
tissue compartments (lung, spleen), a difference attributable to assay modality rather
than data error. Neither database samples a true solid lymph node organ tissue or a
canonical upper-airway/nasal-epithelium site; both gaps are reported explicitly rather
than filled with an unverified proxy tissue (see Limitations below).

## Repository structure

```
data/
  raw/                      IEDB epitope query output, query log, full AF3 submission manifest
  processed/structures/
    benchmarks/             AF3-predicted model_0.cif, 4 benchmark pairs (B1, B2, B4, B5)
    panel/                  AF3-predicted model_0.cif, 35 shortlisted panel pairs
notebooks/                  Steps 3-9: AF3 benchmark, full-panel prediction, groove
                             geometry, PRODIGY affinity, structure-affinity consistency,
                             allele clustering, tissue expression
results/
  tables/                   Supplementary tables S0, S2-S6
  figures/                  Allele structural dendrogram; GTEx/HPA tissue expression heatmaps
docs/
  data_dictionary.md        Column-level description of every table
  pipeline_notes.md         Verification status per step, notebook-to-table mapping, open items
```

## Supplementary tables

| Table | Contents |
|---|---|
| `Table_S0_af3_benchmark.csv` | AF3 vs. crystal structure accuracy (TM-score, RMSD) for 4 benchmark pairs, all confirmed |
| `Table_S2_groove_geometry_concordance.csv` | Contact residues, buried surface area, polar contacts for all 35 panel pairs |
| `Table_S3_prodigy_affinity.csv` | PRODIGY ΔG and Kd for all 35 panel pairs |
| `Table_S4_structure_affinity_correlations.csv` | Spearman correlations, structure vs. affinity, by class |
| `Table_S4_structure_affinity_scatter_data.csv` | Per-pair values underlying the correlations |
| `Table_S5_allele_structural_aggregation.csv` | Allele-level mean groove geometry, by class |
| `Table_S5_allele_pairwise_distances.csv` | Pairwise structural distance, all allele pairs |
| `Table_S6_gtex_hla_expression.csv` | GTEx median TPM per HLA gene per tissue compartment |
| `Table_S6_hpa_hla_expression.csv` | HPA independent tissue RNA-seq per HLA gene per tissue compartment |
| `Table_S6_cross_pipeline_concordance.csv` | GTEx/HPA divergence ratios per gene per tissue |

Full column-level definitions in [`docs/data_dictionary.md`](docs/data_dictionary.md).

## Limitations

Reported explicitly rather than substituted or omitted:

- **No true lymph node tissue** in GTEx or HPA. GTEx's nearest match by name,
  "Cells – EBV-transformed lymphocytes," is a cultured cell line, not lymph node organ
  tissue, and was excluded rather than used as a proxy.
- **No true upper-airway/nasal-epithelium tissue** in GTEx or HPA. GTEx's nearest match,
  "Minor Salivary Gland," is exocrine gland tissue, not airway epithelium, and was
  likewise excluded.
- **HPA column is RNA-seq, not immunohistochemistry.** HPA protein-level IHC data was
  confirmed absent from HPA's current public export surface at the time of analysis
  (checked in both the bulk TSV and the per-gene XML endpoint). `Table_S6_hpa_hla_expression.csv`
  reports HPA's independent tissue RNA-seq instead, making the GTEx/HPA comparison a
  same-modality, cross-pipeline replication rather than a protein-vs-RNA cross-modality
  check.
- **Structure–affinity correlations are exploratory.** Per-class sample sizes (n = 23
  class I, n = 12 class II) are underpowered to detect anything but a strong effect;
  non-significance is reported as a power limitation, not as evidence of no relationship.
- **The class I/class II structural separation is not fully disjoint.** Within-class
  pairwise distances overlap the lower end of the cross-class distance distribution
  (23% of cross-class pairs are smaller than the largest within-class distance), so this
  is reported as a dominant structural signal driving the dendrogram's top-level split,
  not as complete separation.
- **Allele-level clustering uses small per-allele n.** Fifteen of sixteen panel alleles
  contribute exactly 2 pairs each to their allele-level mean (HLA-A\*02:01 contributes 5).
  The class I/class II structural bifurcation is robust to this limitation (it is a
  between-group, not within-allele, comparison), but any finer within-class structure
  should be read as hypothesis-generating.

## Requirements

Pipeline notebooks were run in Google Colab. Key dependencies: `pandas`, `scipy`,
`numpy`, `matplotlib`, `seaborn`, `biopython`, `pymol-open-source-whl`, `prodigy-prot`,
`requests`. AlphaFold3 structure prediction and NetMHCpan/NetMHCIIpan concordance are
manual/local steps with no public batch API; see individual notebooks for submission and
upload instructions.

## Citation

[![DOI](https://zenodo.org/badge/1444590449.svg)](https://doi.org/10.5281/zenodo.24025041)


This repository is archived on Zenodo at the DOI above. Cite this specific release when
referencing the code and data underlying the manuscript.

<!-- Add bioRxiv preprint citation once posted -->

## License

Code (`notebooks/`) is released under the [MIT License](LICENSE).

Data, results, figures, and documentation (`data/`, `results/`, `docs/`) are released
under [CC BY 4.0](DATA_LICENSE) — reuse is permitted with attribution.
