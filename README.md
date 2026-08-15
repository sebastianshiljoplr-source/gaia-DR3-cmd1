# Gaia DR3 Colour–Magnitude Diagram

A reproducible Python analysis of a nearby Gaia DR3 stellar sample. The project builds a quality-controlled colour–magnitude diagram (CMD), identifies broad stellar-population regions, models the observed main-sequence ridge, and tests how parallax precision affects apparent scatter.

## Research question

Can Gaia DR3 astrometry and photometry be used to construct a quality-controlled CMD and quantitatively describe the stellar populations it contains?

The CMD uses Gaia colour \(BP-RP\) and absolute G-band magnitude \(M_G\). It is an observational form of the Hertzsprung–Russell diagram: bluer/hotter stars are generally found to the left, and brighter stars are plotted at the top.

## Key results

| Result | Value |
|---|---:|
| Initial Gaia DR3 snapshot | 5,000 sources |
| Final quality-controlled sample | 4,681 sources |
| White-dwarf candidates | 59 (1.26%) |
| High-luminosity candidates | 189 (4.04%) |
| Main-sequence working sample | 4,211 (89.96%) |
| Third-order ridge-fit RMSE | 0.120 mag |
| Robust observed scatter around the ridge | 0.491 mag |
| Faint-side residual outliers | 157 |
| Faint-side outliers remaining at parallax S/N > 10 | 20 |

The sensitivity test shows that parallax precision materially contributes to the faint-side broadening of the observed main sequence: tightening the parallax signal-to-noise threshold from >5 to >10 retains only 20 of the original 157 faint-side outliers.

## Project structure

```text
.
├── 02_gaia_data_analysis_clean.ipynb   # Complete, guided analysis
├── data/
│   └── gaia_dr3_initial_5000.ecsv      # Fixed Gaia input snapshot
├── report/
│   └── Gaia_DR3_CMD_Report.pdf         # 6-page scientific summary report
└── README.md
```

## Data source and selection

The original sample was queried from Gaia DR3 table `gaiadr3.gaia_source` through the Gaia Archive. The project uses the stored ECSV snapshot by default, so that rerunning the analysis does not depend on a live query returning the same 5,000-row subset.

```sql
SELECT TOP 5000
    source_id,
    parallax,
    parallax_error,
    phot_g_mean_mag,
    phot_bp_mean_mag,
    phot_rp_mean_mag,
    phot_bp_rp_excess_factor,
    ruwe,
    visibility_periods_used
FROM gaiadr3.gaia_source
WHERE parallax > 2
  AND parallax_over_error > 5
  AND ruwe < 1.4
  AND visibility_periods_used >= 8
  AND phot_g_mean_mag IS NOT NULL
  AND phot_bp_mean_mag IS NOT NULL
  AND phot_rp_mean_mag IS NOT NULL
```

The query-level criteria select a nearby sample (parallax > 2 mas, approximately within 500 pc) with basic astrometric quality requirements. A post-query photometric-quality cut retains sources with:

\[
-0.08 < C^* < 0.20,
\]

where \(C^*\) is the colour-corrected Gaia BP/RP flux-excess factor.

## Method

For every retained Gaia source, the notebook calculates:

\[
d\,(\mathrm{pc})=\frac{1000}{\varpi\,(\mathrm{mas})}
\]

\[
BP-RP=m_{BP}-m_{RP}
\]

\[
M_G=m_G-5\log_{10}(d)+5.
\]

It then:

1. Creates the final CMD and density representation.
2. Selects broad white-dwarf and high-luminosity candidate regions.
3. Defines a main-sequence working region.
4. Estimates a median main-sequence ridge in colour bins.
5. Fits a third-order polynomial to that ridge.
6. Analyses residuals and repeats the outlier check at stricter parallax precision.

## Run the notebook

### Requirements

- Python 3.10 or later
- Jupyter Notebook or JupyterLab
- NumPy
- Matplotlib
- Astropy

Install the packages if needed:

```bash
pip install jupyter numpy matplotlib astropy
```

Open `02_gaia_data_analysis_clean.ipynb` from the project folder and run:

```text
Kernel → Restart Kernel and Run All Cells
```

The notebook expects the data file at:

```text
data/gaia_dr3_initial_5000.ecsv
```

If you keep the data somewhere else, update the `DATA_PATH` line in the first code cell to that file's location.

## Important limitations

- Direct parallax inversion is used only because the sample has a parallax S/N > 5; it should not be treated as a general distance estimator for low-precision parallaxes.
- No interstellar extinction or reddening correction is applied.
- The white-dwarf and high-luminosity regions identify **candidates**, not confirmed stellar classifications.
- The main-sequence polynomial is an empirical description of this sample, not a universal stellar-evolution relation.
- The input is a fixed, nearby 5,000-source snapshot. The population fractions are not estimates for the entire Milky Way.

## Reproducibility notes

The ECSV file is included to preserve the exact input sample used for the reported results. The notebook keeps the original ADQL query for provenance but does not rerun it automatically, because an unordered `TOP 5000` live query can return a different subset in the future.

## Data acknowledgement

This project uses data from the [Gaia mission](https://www.cosmos.esa.int/gaia) and the [Gaia Archive](https://gea.esac.esa.int/archive/).
