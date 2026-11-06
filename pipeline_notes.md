# Pipeline notes: verification status and open items

This file tracks, per pipeline step, what has been verified against real executed output
and what still needs attention before submission. It is a working checklist, not a
methods section.

## Step-by-step verification status

| Step | What it does | Status |
|---|---|---|
| 2 | IEDB epitope query, QC, 932-pair pool | Verified — `step2_iedb_epitopes.csv`, `step2_iedb_query.log` are real query output |
| 3 | AF3 benchmark vs 4 crystal structures (reduced from 5) | Verified — all 4 TM-score/RMSD values confirmed, including recomputation of 2 against corrected references — see **Item 1** below |
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

### 1. Step 3 benchmark: B5 corrected to PDB 1DLH (fixed); B2/B4 re-referenced and recomputed; B3 dropped

**B5** — `notebooks/MS3_step3_af3_benchmark.ipynb`'s `BENCHMARK_PAIRS` list previously defined B5
against PDB 1KLU. The real executed output (`Table_S0_af3_benchmark.csv`) shows B5 was
actually benchmarked against 1DLH (HLA-DR1/HA 306-318; TM-score 0.9902, RMSD 0.433 Å,
resolution 2.8 Å, 1994 deposition) — there is no 1KLU result anywhere in the data. The
notebook has been corrected so `BENCHMARK_PAIRS` reads 1DLH, with a note in the cell
documenting the correction, so the published code now matches the published numbers.

**B2 and B4 — re-referenced to the correct crystal structures; TM-score/RMSD recomputed.**
Direct inspection of the predicted `.cif` files confirmed the AF3 structures themselves are
correct (right HLA heavy-chain sequence, right peptide in chain C). Only the external
crystal-structure accessions used for TM-score/RMSD comparison were wrong:

- B2 (NLVPMVATV/HLA-A\*02:01): corrected 1T1W → **3GSO** (Gras et al., *J Immunol*
  2009;183:430–437; 1.60 Å). 1T1W is a real structure, but of HLA-A2 with an HIV-1 Gag
  SL9 variant peptide (SLYNTVATL) — unrelated to this complex.
- B4 (RPHERNGFTVL/HLA-B\*07:02): corrected 1W0V → **3VCL** (Brennan et al., *J Immunol*
  2012;188:2742–2748; 1.70 Å; allele annotated generically as "HLA-B7" on this PDB
  entry rather than the resolved B\*07:02 subtype). 1W0V is a real structure, but of
  HLA-B\*2705 with an unrelated 9-residue self-peptide — unrelated to this complex.

`Table_S0_af3_benchmark.csv` now carries the corrected PDB IDs and real recomputed values
for B2 and B4: **B2 TM-score 0.9922, RMSD 0.535 Å; B4 TM-score 0.9629, RMSD 1.172 Å.**
Both reference coordinate files (3GSO.cif, 3VCL.cif, downloaded directly from RCSB) were
confirmed by direct chain-sequence inspection to contain the correct heavy-chain/peptide
complex before alignment (3GSO: chain A = HLA-A heavy chain, chain B = β2-microglobulin,
chain P = NLVPMVATV; 3VCL: chain A = HLA-B7 heavy chain, chain B = β2-microglobulin,
chain C = RPHERNGFTVL — both exactly matching the claimed complex). Superposition was run
two independent ways as a cross-check: PyMOL CEAlign (window size 8, matching the method
used for B1/B5) on the heavy chain alone, and TM-align (via the `tmtools` Python binding)
on Cα coordinates. The two methods' RMSDs agree closely (B2: 0.535 Å CEAlign vs 0.543 Å
TM-align; B4: 1.172 Å CEAlign vs 1.195 Å TM-align), and the TM-score reported is TM-align's
score normalized to the reference (crystal-structure) chain length, the conventional
choice when scoring a predicted structure against a fixed experimental target. The CEAlign
RMSD is the value carried into `rmsd_angstrom` for consistency with B1/B5's `rmsd_method`
column (`cealign_w8`).

**B3 — dropped (HLA-B\*07:02/RPRGEVRFL, previously mis-cited against 3MGT).** 3MGT is a
real structure, but of HLA-A\*02:01 with an H5N1 influenza HA peptide — unrelated to
either the allele or peptide this benchmark claimed. No crystal structure of RPRGEVRFL
bound to any HLA allele could be found on RCSB, PDBe, or PDBj after a thorough search
(including sequence-adjacent near-misses: 6VMX is RPPIFIRRL, not RPRGEVRFL; the related
HSV-2 VP22 15-mer PMRARPRGEVRFLHY restricted by HLA-B\*07 appears in TCR-repertoire
literature but no deposited structure of the minimal 9-mer could be confirmed). The
benchmark set is now 4 structures (B1, B2, B4, B5), not 5. If a genuine RPRGEVRFL
structure is later confirmed, reinstate it as an independent 5th benchmark rather than
reusing 3VCL, which already serves as the B4 reference.

### 2. Step 8 clustering: within-class control added for the chain-count question

`Table_S5_allele_structural_aggregation.csv` shows zero overlap between class I and
class II allele means on every feature (BSA, contact count, polar contacts). Class II
complexes have two HLA chains contributing to the modelled interface (DRA+DRB1); class
I complexes have one (the heavy chain only). A larger structural interface for class II
is therefore expected from chain count alone, independent of any genuine allele-level
signal the clustering is meant to detect.

`notebooks/MS3_step8_allele_clustering.ipynb` includes a Part E that re-runs Ward
clustering **separately within each class** (re-standardized within that class only).
That cell has not been executed (no cached output) — the numbers below were computed
directly against `Table_S5_allele_structural_aggregation.csv` outside the notebook and
still need to be reproduced by actually running Part E before submission.

On the single full-panel z-score (the same standardization used for the combined
dendrogram and the "four largest distances" claim), within-class pairwise distances are
small relative to cross-class distances but **not negligible**:

| | n pairs | median | range |
|---|---|---|---|
| Within class I | 45 | 0.734 | 0.189–2.152 |
| Within class II | 15 | 1.168 | 0.451–2.621 |
| Cross-class | 60 | 3.258 | 1.566–5.420 |

The four largest pairwise distances in the full matrix are still all cross-class, which
supports the headline bifurcation claim. But 14 of the 60 cross-class pairs (23%) fall
below the largest within-class distance (2.621) — so **"complete separation" / "zero
overlap whatsoever" is too strong** for the manuscript text. The accurate framing: class
I/class II separation is the dominant structural signal in this feature space, strong
enough to drive the combined dendrogram's top-level split, but the two distributions
are not disjoint, and within-class allelic variation is real, not zero.

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

### 4. NetMHCpan rank_el data exists for all 35 pairs but was never reported — new finding, needs independent re-verification

`Table_S2_groove_geometry_concordance.csv` carries a populated `netmhc_rank_el` column
(NetMHCpan/NetMHCIIpan eluted-ligand percentile rank) for all 35 panel pairs — this was
never mentioned in the README, the manuscript draft, or any prior session note, and was
only found by checking the table schema directly while assembling citations. It is real
data sitting in the repository, not something newly computed.

Summary statistics (computed directly from the table, outside any notebook):
- 28/35 pairs (80%) meet the standard NetMHCpan ≤2.0%% rank_el threshold for predicted
  binding; 20/35 (57%) meet the ≤0.5%% strong-binder threshold.
- Spearman correlation between `netmhc_rank_el` and PRODIGY `delta_g_kcal_mol`: pooled
  across both classes, ρ=0.014 (p=0.938, n=35) — no relationship. Within class I alone,
  ρ=0.586 (**p=0.0033, n=23**) — significant. Within class II alone, ρ=0.151 (p=0.640,
  n=12) — not significant.

The class I result is a genuine, independently interesting finding: sequence-based
(NetMHCpan) and structure-based (PRODIGY) binding predictions agree within class I even
though neither correlates with groove geometry features (Item in Table_S4). It did not
hold in class II or when classes were pooled. **This has not been independently
re-verified by re-running the correlation inside a notebook** — it was computed once,
directly against the CSV, during manuscript drafting. Re-run it in
`notebooks/MS3_step7_structure_affinity_consistency.ipynb` (or a new cell) before relying
on it for submission, and decide whether `netmhc_ic50_nm` (currently empty for all rows)
should be populated or the column dropped from the data dictionary.

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
