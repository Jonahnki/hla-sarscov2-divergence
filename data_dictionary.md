# Data dictionary

Column-level description of every table in `results/tables/` and the raw files in
`data/raw/`. Units are given where not obvious from the column name.

## data/raw/

### step2_iedb_epitopes.csv
Raw epitope records returned by the IEDB PostgREST API query (Step 2), before
deduplication or coverage-aware shortlisting.
- `allele` — HLA allele, IEDB nomenclature
- `peptide` — epitope sequence
- `donor_count`, `ic50` — IEDB-reported immunogenicity fields; uniformly null/zero across
  this dataset (not populated for the queried epitope set), which is why ranking fell back
  to record-count-based evidence strength rather than a donor/IC50-weighted score
- other columns as returned verbatim by the IEDB API

### step2_iedb_query.log
Per-allele query statistics: raw records returned vs records passing basic QC
(length, valid amino-acid alphabet, non-duplicate) for each of the 16 panel alleles.
Raw range 25–1,924; passed range 3–265.

### step4_final_manifest.csv
Submission/retrieval status for every one of the 932 deduplicated (allele, peptide) pairs
considered during Step 4, not just the 35 shortlisted. `status` values include
`structure_ready`, `submitted`, `pending`. Only the 35 shortlisted pairs were carried
forward past this step; the remainder represent the full candidate pool before the
coverage-aware selection (`MIN_PER_ALLELE=2`, `TARGET_TOTAL=35`).

## results/tables/

### Table_S0_af3_benchmark.csv
AF3 structure-prediction accuracy against 4 independent crystal structures (B1, B2, B4, B5 —
B3 dropped, see note below).
- `benchmark_id` — B1, B2, B4, B5
- `allele`, `peptide` — benchmark complex identity
- `pdb_id` — reference crystal structure (1OGA, 3GSO, 3VCL, 1DLH)
- `tm_score` — template modelling score, AF3 model vs crystal structure
- `rmsd_angstrom` — Cα RMSD, Å
- `resolution_angstrom`, `deposit_year` — reference structure metadata

Note: B5 is 1DLH (TM-score 0.9902, RMSD 0.433 Å). B2 and B4 were re-referenced from
1T1W and 1W0V (both real structures, but of different allele/peptide complexes than
claimed) to 3GSO and 3VCL; TM-score/RMSD have been recomputed against the corrected
references (B2: TM-score 0.9922, RMSD 0.535 Å; B4: TM-score 0.9629, RMSD 1.172 Å),
confirmed by two independent superposition methods (PyMOL CEAlign and TM-align) in
close agreement. B3 (HLA-B\*07:02/RPRGEVRFL vs 3MGT, also a wrong reference) was
dropped — no genuine RPRGEVRFL structure could be found. See `pipeline_notes.md`
Item 1 and `notebooks/MS3_step3_af3_benchmark.ipynb` for the full correction.

### Table_S2_groove_geometry_concordance.csv
Peptide-binding groove geometry for all 35 panel pairs, computed from AF3 models in
PyMOL.
- `pair_id`, `allele`, `peptide`, `hla_class` (I/II)
- `n_contact_residues` — HLA residues within 4.5 Å of the peptide
- `buried_surface_area_A2` — BSA, Å², SASA-delta method
- `n_polar_contacts` — hydrogen-bond-capable contacts
- `mean_plddt` — mean AF3 per-residue confidence (pLDDT) across the groove, used as a
  structural-confidence proxy
- `netmhc_rank_el` — NetMHCpan/NetMHCIIpan eluted-ligand percentile rank, populated for
  all 35 pairs; not previously reported in the manuscript or README (see
  `pipeline_notes.md` Item 4). Correlates significantly with `Table_S3`'s
  `delta_g_kcal_mol` within class I (Spearman ρ=0.586, p=0.0033, n=23) but not within
  class II (ρ=0.151, p=0.640, n=12) or pooled (ρ=0.014, p=0.938, n=35)
- `netmhc_ic50_nm` — empty for all 35 rows; not populated in this run

Class I entries use the HLA heavy chain only; class II entries use HLA-DRA + HLA-DRB1
jointly (locus-aware chain selection, consistent with Table_S3).

### Table_S3_prodigy_affinity.csv
PRODIGY binding-affinity estimates on the same 35 AF3 models.
- `pair_id`, `allele`, `peptide`, `hla_class`
- `chain_selection` — PRODIGY chain groups used (`"A,B"` vs peptide chain `"C"` for class
  II; `"A"` vs `"C"` for class I)
- `delta_g_kcal_mol` — predicted ΔG, kcal/mol (more negative = tighter binding)
- `kd` — predicted dissociation constant
- `status` — all 35 rows `ok` in this dataset; any future `flagged` status would mark a
  physically implausible positive ΔG

### Table_S4_structure_affinity_correlations.csv
Spearman correlation between each groove-geometry feature (contacts, BSA, polar
contacts) and PRODIGY ΔG, computed separately within class I and within class II
(not pooled, to avoid a class-split-driven trivial result).
- `feature`, `hla_class`, `n_pairs`, `spearman_rho`, `p_value`
- All 6 rows (3 features × 2 classes) non-significant, p = 0.1612–0.909

### Table_S4_structure_affinity_scatter_data.csv
Per-pair values underlying the Table_S4 correlations (one row per pair per feature),
for figure reproduction.

### Table_S5_allele_structural_aggregation.csv
Allele-level means of groove-geometry features (BSA, contacts, polar contacts),
aggregated across each allele's shortlisted pairs. Input to the Step 8 hierarchical
clustering.
- `allele`, `hla_class`, `n_pairs`, mean of each Table_S2 feature

Class II allele means (BSA 2166-2317 Å², contacts 35-41.5, polar 103.5-119.5) do not
overlap Class I allele means (BSA 1770-1855 Å², contacts 28.5-33.5, polar 90-104) on any
single feature — this is still true and separate from the pairwise-distance clustering
question below. See `pipeline_notes.md` Item 2 for the pairwise Euclidean-distance
picture, which is more nuanced: within-class distances and cross-class distances are not
fully disjoint (23% of cross-class pairs fall below the largest within-class distance),
even though no single feature's allele-level means overlap between classes.

### Table_S5_allele_pairwise_distances.csv
Pairwise Euclidean distance between every pair of alleles in z-scored feature space
(input to Ward-linkage clustering). The four largest distances in the full matrix are
all class-I-vs-class-II allele pairs.

### Table_S6_cross_pipeline_concordance.csv
Per-gene comparison of GTEx and HPA expression values across shared tissues, with the
computed fold-divergence ratio per tissue.

### Table_S6_gtex_hla_expression.csv
GTEx (`clusteredMedianGeneExpression` endpoint) median expression per HLA gene across
54 tissue entries. No true lymph node organ tissue or upper-airway/nasal site is present
in GTEx's tissue panel (nearest entries — EBV-transformed lymphocytes, minor salivary
gland — were explicitly excluded as non-equivalent; see `pipeline_notes.md`).

### Table_S6_hpa_hla_expression.csv
Human Protein Atlas tissue RNA-seq (`assayType="tissue"`) expression per HLA gene.
This is RNA-seq, not protein-level immunohistochemistry — HPA's IHC export was
confirmed unavailable for this gene set at the time of the pipeline run (absent from
both the bulk TSV and per-gene XML). This table is a same-modality replication of the
GTEx result, not a protein-vs-RNA cross-modality check. See `pipeline_notes.md`.

## results/figures/

- `Figure_allele_structural_dendrogram.png` — Ward-linkage clustering of alleles on
  z-scored Table_S5 features
- `Figure_gtex_hla_tissue_expression.png` — GTEx tissue expression heatmap, per HLA gene
- `Figure_hpa_hla_tissue_expression.png` — HPA tissue RNA-seq expression heatmap, per HLA
  gene
