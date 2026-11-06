# A Hybrid ChemBERTa–Morgan XGBoost Ensemble for Computational Multi-Endpoint Toxicity Prioritization of *Blumea balsamifera* Compounds

Code, result archives and figures accompanying the article submitted to
*Statistics, Optimization & Information Computing* (manuscript 4593-SOIC).

- **Archived version (cite this):** `TODO — Zenodo version DOI`
- **Article DOI:** `TODO — assigned on publication`

Everything in this deposit was produced from a single curated dataset of
**7,817 Tox21 structures**. Any intermediate count other than 7,817 does not
belong to this release.

---

## 1. Contents

| Folder | Files | Contents |
|---|---|---|
| `hasil/` | 44 | Every result file behind the tables and figures |
| `kode/` | 7 | The pipeline as notebook and script, the baseline script, two figure scripts, and the run order |
| `data/` | 4 | The LC-MS annotation of the application set, the 31 structures with canonical SMILES, the consolidated supplementary table, and the thresholds |

The Tox21 release itself is not redistributed. It is publicly available
through the Tox21 Data Challenge and PubChem.

---

## 2. Curation

| Step | Removed | Remaining |
|---|---|---|
| Tox21 release, 12 endpoints | — | 7,831 |
| Invalid or unparsable SMILES | 8 | 7,823 |
| Structures overlapping the application set, on canonical SMILES computed without stereochemistry | 6 | **7,817** |

The six exclusions are three structures identical to an application compound on
isomeric canonical SMILES — (+)-limonene, cuminaldehyde and quercetin — and
three stereoisomers whose Morgan fingerprints are identical to an application
compound, since the fingerprints used here do not encode chirality.

`hasil/Audit_Konsistensi.csv` checks this count at every pipeline stage and
reports `True` on all 25 rows.

---

## 3. Protocol and estimator

**Partition.** Three repetitions of five-fold Bemis–Murcko scaffold
cross-validation, giving 15 outer folds. Within each outer fold the training
data are split again by scaffold into a base training set and a calibration
set. Repetition seeds are 42, 73 and 101.

**Estimator — read this before using `hasil/`.** Every metric in the article is
computed per endpoint on out-of-fold probabilities averaged over the three
repetitions, then averaged over the 12 endpoints, with the standard deviation
taken across endpoints. Two alternatives are archived and are *not* used in the
article text. For the adopted ensemble the three give:

| Estimator | ROC-AUC | File |
|---|---|---|
| **Article estimator** — repeat-averaged probabilities, then per endpoint | **0.793** | `Significance.csv`, columns `value_a` / `value_b` |
| Per repetition, then averaged | 0.755 | `Perbandingan_Model.csv` |
| All endpoints pooled into one ranking | 0.819 | `Perbandingan_Model_Pooled.csv` |

Reading an article figure off the wrong file is the single easiest mistake to
make in this archive. Seven files are computed on the per-repetition estimator
and therefore return values **lower** than the article for the same models:

| File | Example of the difference |
|---|---|
| `Perbandingan_Model.csv` | Ensemble 0.755, article 0.793 |
| `Metrik_Per_Repeat.csv` | Ensemble 0.755, article 0.793 |
| `Variance.csv` | between-endpoint SD 0.044, article Table 4 gives 0.050 |
| `Garis_Dasar_Ringkasan.csv` | bagged trees 0.734, article 0.772 |
| `Garis_Dasar_Per_Repeat.csv` | logistic regression 0.611, article 0.655 |
| `baseline_utama_per_repeat.csv` | linear SVM 0.617, article 0.653 |
| `baseline_utama_ringkasan.csv` | same basis as the two above |

`Perbandingan_Model_Pooled.csv` differs in the other direction, returning
higher values (ensemble 0.819). None of these are errors: they are the two
alternative estimators, archived deliberately. Every article figure comes from
`Significance.csv` and `baseline_utama_per_endpoint.csv`, which are on the
article estimator. The one exception inside the article is the variance
decomposition in Section 3.1, which necessarily uses per-repetition metrics and
says so.

---

## 4. Where each table comes from

| Table | Source in `hasil/` |
|---|---|
| 1. Dataset summary | `Ringkasan_Eksklusi.csv`, `Audit_Eksklusi.csv` |
| 2. Molecular representations | descriptive; parameters set in `kode/` |
| 3. Main XGBoost parameters | configuration in `kode/`; checked by `Nested_CV.csv` |
| 4. Model performance, Block 1 | `Significance.csv` (`value_a` / `value_b`) |
| 4. Model performance, Block 2 | `Ablasi_Ringkasan.csv` |
| 5. Combination strategies | `Significance.csv` for the *p*-values, `Ece.csv` for the Brier scores |
| 6. Endpoint characteristics | `Endpoint_Characteristics.csv`, `Structural_Coherence.csv` |
| 7. Endpoint-specific thresholds | `Threshold.csv`; prevalence from `Endpoint_Characteristics.csv` |
| 8. Top-ranked compounds | `Probabilitas_Blumea.csv`, `Threshold.csv`, `Stabilitas_Ranking.csv` |

---

## 5. Where each figure comes from

The figure files themselves are not deposited; they are in the article. This
table records how each was produced.

| Figure | Script | Data |
|---|---|---|
| 1. Workflow | `kode/roadmap.py` | counts from Section 2 |
| 2. Active/inactive distribution | `TODO` | `Jumlah_Label_Per_Titik_Akhir.csv` |
| 3. Endpoint ROC curves | `kode/buat_4_gambar_sisa.py` | out-of-fold ensemble predictions |
| 4. Precision–recall curves | `kode/buat_4_gambar_sisa.py` | out-of-fold ensemble predictions |
| 5. Normalized confusion matrices | `TODO` | out-of-fold predictions, `Threshold.csv` |
| 6. Reliability curves | `kode/buat_4_gambar_sisa.py` | out-of-fold predictions, `Ece.csv` |
| 7. ROC-AUC with bootstrap CIs | `kode/buat_4_gambar_sisa.py` | out-of-fold ensemble predictions |
| 8. Threshold analysis (A, B) | `TODO` | `Threshold.csv`, `Threshold_Sensitivity.csv` |
| 9. Multi-endpoint priority score | `TODO` | `Probabilitas_Blumea.csv`, `Stabilitas_Ranking.csv` |
| 10. Calibrated probability profiles | `TODO` | `Probabilitas_Blumea.csv` |
| 11. Probability margins | `TODO` | `Probabilitas_Blumea.csv`, `Threshold.csv` |
| 12. Applicability-domain strata | `TODO` | `Ad_Stratified.csv` |
| 13. Maximum Tanimoto similarity | `TODO` | `Applicability_Domain.csv` |

`kode/buat_4_gambar_sisa.py` reads the ensemble out-of-fold predictions from
the pipeline checkpoint and writes `oof_ensemble.csv.gz` alongside the four
figures it produces. That file is not archived here because it is regenerated
by the script.

---

## 6. The main result files

| File | Contents |
|---|---|
| `Significance.csv` | Every model comparison: bootstrap difference, 95% CI, Holm-corrected *p*, DeLong *z* and *p*, and both models' values under the article estimator |
| `Audit_Konsistensi.csv` | Count checks at each pipeline stage; all 25 rows read `True` |
| `Endpoint_Characteristics.csv` | Labels, actives, prevalence, scaffolds and missingness per endpoint |
| `Structural_Coherence.csv` | Nearest-active similarity, model-free baseline, cross-panel co-activity, per-endpoint performance |
| `Threshold.csv` | Youden thresholds per endpoint, model and repetition |
| `Threshold_Sensitivity.csv` | Balanced accuracy, sensitivity, specificity and MCC at threshold multipliers 0.6 to 1.5 |
| `Ece.csv` | Expected calibration error, Brier score, mean predicted probability against observed prevalence |
| `Ad_Stratified.csv` | Performance by quartile of similarity to the training set |
| `Applicability_Domain.csv` | Leverage, maximum Tanimoto, distance to centroid and domain flag, per compound |
| `Stabilitas_Ranking.csv` | Priority score and rank under each of the three seeds |
| `Probabilitas_Blumea.csv` | Calibrated probability of each of the 31 compounds at each of the 12 endpoints |
| `Nested_CV.csv` | Nested hyperparameter search on SR-ARE and SR-MMP, five outer folds each |
| `REVISI_SOIC_HASIL_LENGKAP.xlsx` | The same content as one workbook, one sheet per file |

The remaining files hold the supporting analyses: leakage
(`Kebocoran_*.csv`), the chemically distant holdout
(`Holdout_Kimia_Terjauh.csv`), score decomposition
(`Score_Decomposition*.csv`), alternative endpoint weighting
(`Weighting*.csv`), seed agreement (`Kesesuaian_Seed.csv`), false negatives by
scaffold (`Blind_Spots.csv`, `False_Negative.csv`), calibration effects
(`Efek_Kalibrasi.csv`), dataset-characteristic correlations
(`Endpoint_Correlations.csv`, `Structural_Correlations.csv`), the ablation
(`Ablasi*.csv`), the exclusion audit (`Audit_Eksklusi.csv`,
`Ringkasan_Eksklusi.csv`), label counts
(`Jumlah_Label_Per_Titik_Akhir.csv`) and the combined significance summary
(`Significance_Summary.csv`).

---

## 6b. The application set

| File | Contents |
|---|---|
| `data/blumea_31_structures.csv` | The 31 compounds with molecular formula and canonical SMILES, as used by the pipeline |
| `data/blumea_31_supplementary.csv` | One row per compound: SMILES, priority score and rank, applicability-domain measures, the 12 calibrated probabilities, and the margin of each probability over its endpoint threshold |
| `data/endpoint_thresholds.csv` | The ensemble Youden threshold applied at each endpoint |
| `data/ini_saja.xlsx` | The LC-MS annotation the pipeline reads: 129 records, 31 unique compounds, with formula, canonical SMILES and physicochemical descriptors |

Together these are the supplementary material the article describes.

**Data note.** For two compounds the `Formula` column of the source LC-MS
annotation and the `Canonical SMILES` column do not agree:
*3,5,3',4'-tetrahydroxy-7-methoxyflavone* (formula C16H12O7, SMILES C17O7) and
*dihydroquercetin-7,4'-dimethylether* (formula C17H16O7, SMILES C15O7). The
modelling used the SMILES, which is also what the physicochemical descriptors
in the source annotation were computed from.

---

## 7. Before publishing this deposit

Four things are missing and only the authors can supply them.

**Eight figure scripts.** Section 5 above has eight `TODO` entries. The scripts
exist; add them to `kode/` and replace each `TODO` with its filename.

**Environment.** Add a `requirements.txt` with pinned versions of Python,
rdkit, xgboost, scikit-learn, transformers, torch, numpy, pandas and
matplotlib, and the commands that run the pipeline end to end.

Two further cautions. Clear notebook outputs before uploading: saved outputs
can carry figures from earlier runs and local file paths. And once a Zenodo
record is published its files are immutable, so settle the contents first — the
DOI can be reserved beforehand and used in the manuscript without publishing.

---

## 8. Scope of what these results support

The 12 Tox21 endpoints are specific *in vitro* assay readouts, not measures of
toxicity in an organism. The priority score is one hundred times the mean
calibrated probability across the panel, and is a model-implied prioritization
measure rather than a validated activity frequency; its reading is conditional
on endpoint calibration quality and on the compound lying within the
applicability domain. All evaluation is internal to Tox21: the chemically
distant holdout bounds extrapolation but is not independent external
validation. These limits are stated in the article and apply equally to
anything derived from this deposit.

---

## 9. Licence and citation

- `kode/`: `TODO — MIT or Apache-2.0`
- `hasil/`, `data/`: `TODO — CC-BY-4.0`

```
TODO — citation once the article has a DOI
```

**Corresponding author:** `TODO — name and email`
