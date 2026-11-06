# Open Cluster Membership Project: Progress Log

**Goal:** Determine open-cluster membership with (a) a classical Bayesian proper-motion (PM) mixture model and (b) DBSCAN, then cross-match the two (as in Alfonso & García-Varela 2023, "AG23"). Tooling: Python in Google Colab, Gaia DR3. Targets: **Pleiades (Melotte 22 / M45)** and **Beehive (Praesepe / NGC 2632 / M44)**.

## Reference key
| Short | Paper |
|---|---|
| **AG23** | Alfonso & García-Varela 2023, A&A 677, A163 (Pleiades, Praesepe, Blanco 1; PM model + DBSCAN). Main blueprint |
| **CG18** | Cantat-Gaudin+ 2018 (Gaia DR2, UPMASK, 1229 clusters) |
| **Deb22** | Deb, Baruah & Kumar 2022, MNRAS 515, 4685 (kNN + Mahalanobis distance + 2-component GMM) |
| **Gao19** | Gao 2019, PASP 131, 044101 (Pleiades, GMM + Random Forest) |
| CG-Ginard18 | Castro-Ginard+ 2018, A&A 618, A59. **Not yet provided**; AG23 adapts its DBSCAN recipe |

---

## Status
- [x] Phase 1 done: Pleiades and Beehive datasets downloaded, loaded in Colab as DataFrames.
- [x] Dropped null rows and kept the core columns: ra, dec, parallax, pmra, pmdec.
- [ ] **Phase 2 (next): exploration and region selection** (see plan below)

---

## Plan (phases now cross-referenced to the literature)

### Phase 1: Environment and data (DONE)
- AG23 Sec. 2 + Appendix A: 10 deg cone search, quality cuts (ruwe < 1.4, flux_over_error > 10 in G/BP/RP, visibility_periods_used > 8, astrometric_excess_noise < 1, parallax_over_error > 10, parallax > 2 mas, non-null/non-zero pm).
- Gao19 Sec. 2.1: 6.5 deg radius, parallax > 3.33 mas (< 300 pc), |PM| < 60 mas/yr, requires G/BP/RP present. Gives 31,462 Pleiades stars.
- CG18 Sec. 2.1, 3.2.1: uses only G < 18, cone radius = 2x catalogued diameter, parallax within 0.5 mas of expected.
- Deb22 Sec. 2: Gaia DR3; searches within 5-20 arcmin (small) and up to 3 deg (large) for M67.
- **Lesson:** every paper applies parallax and PM cuts at query time, so the null-heavy columns are not what is being used for clustering.

### Phase 2: Exploration and region selection (CURRENT)
Plots to make (details in Entry 2 below):
1. PM plane (VPD), density-aware. Refs: AG23 Fig. 1; Deb22 Fig. 2 (middle top); Gao19 Figs. 2-3 (right panels).
2. PM marginal histograms (pmra, pmdec) with Gaussian fit. Refs: AG23 Sec. 2 (3 sigma window); Deb22 Fig. 2 (bottom row), Sec. 3.1.
3. Parallax histogram. Refs: Deb22 Fig. 2 and Fig. 3; Gao19 Fig. 5.
4. Sky positions (ra/dec) and parallax vs angular distance from centre. Refs: Gao19 Figs. 4 and 8 (pi-R plane); CG18 Fig. 8 (sky map).
5. Parallax vs PM scatter (depth effect). Ref: Gao19 Sec. 3, Fig. 6, Eqs. 7-8.
6. Region selection: 3 sigma window around cluster PM. Ref: AG23 Sec. 2. Contrast: CG18 Sec. 3.2.2 does NOT restrict PM for Melotte 22 (Pleiades) or NGC 2632 (Praesepe).
7. Preview of k-nearest-neighbour distance histogram. Refs: Deb22 Fig. 2 (left top), AG23 Fig. 2.

### Phase 3: Bayesian PM mixture model
- AG23 Sec. 3.1 Eqs. 1-6; Sec. 4.1, Table 1 (parameters: Pleiades n_c 0.48, sigma_c 1.11, mu_c (19.95, -45.41); Praesepe n_c 0.30, sigma_c 0.99, mu_c (-35.98, -12.86)); Appendix B (emcee: 100 walkers, 5000 iterations, Gelman-Rubin check).
- Original method: Vasilevskis+ 1958, Sanders 1971 (cited in AG23, Deb22, Gao19; not provided).
- Related mixture-model views for comparison: Deb22 Sec. 3.3 (1D 2-component GMM via EM, Eqs. 1-2, membership = responsibility); Gao19 Sec. 2.2 (2-component GMM in 4D, Eqs. 1-4, EM).
- Note: AG23 field model is elliptical with correlation rho; cluster model is **circular**. Gao19 (Sec. 3, Fig. 2) notes the cluster is elongated in the PM plane due to depth. Test whether a circular cluster Gaussian is adequate.

### Phase 4: DBSCAN
- AG23 Sec. 3.2: 5D (ra, dec, pmra, pmdec, parallax), z-score standardised, minPts = 8 (k = 7), eps = (eps_rand + eps_kNN)/2 with 30 resamplings; eps about 0.404 (Pleiades), 0.371 (Praesepe). Fig. 2.
- Recipe origin: CG-Ginard18 (needed).
- Related outlier/density idea: Deb22 Sec. 3.1 (average kNN distance d_NN in 3D (pi, pmra, pmdec); Table 1 gives cutoffs per cluster, Fig. 2).
- Related unsupervised-with-significance idea: CG18 Sec. 3.1 UPMASK (k-means groups + minimum-spanning-tree veto, probabilities from 10 redraws).
- Sensitivity: AG23 Sec. 4.2 says minPts choice within 5-9 changes member counts little.

### Phase 5: Cross-match and validate
- AG23 Sec. 4 (cross-match PM model and DBSCAN; Table 2: Pleiades PM 1041, DB 1026, cross 958; Praesepe 795 / 885 / 744).
- Validation on independent dimensions: Deb22 Sec. 3 (CMD, PM vector plot), Fig. 3; AG23 Fig. 5 (CMD with BASE-9 isochrone); CG18 Appendix A (sky + CMD per cluster).
- Compare with Gao19 Sec. 2.4 (Pleiades 1454 members, P_RF >= 0.6, includes LB 1497 white dwarf); CG18 (992 Pleiades members, G < 18).

### Phase 6: Cluster properties (optional)
- AG23 Sec. 3.3, 4.3: King profile fit; Sec. 4.4 spatial distribution; Sec. 4.5 age/metallicity (BASE-9).
- Deb22 Sec. 4: King fit via emcee (2000 walkers, 1000 iterations).
- Gao19 Sec. 3: distance via Monte Carlo of median parallax (136.0 pc); Sec. 4: King profile, limiting radius, mass segregation.
- CG18 Sec. 4.2: maximum-likelihood distance; parallax zero point (+0.029 mas for DR2).

### Phase 7: Stretch goals
- Harder cluster with overlapping peaks (Deb22 Sec. 5: NGC 2539, IC 4651, NGC 2141, Berkeley 18).
- Compare with Deb22 Mahalanobis + GMM approach.

---

## Entry 1 (2026-11-06): initial plan
Created the phased plan; chose AG23 as blueprint; flagged questions (cluster choice, replicate vs new, Castro-Ginard 2018 request).

## Entry 2 (2026-11-07): Phase 2 guidance and literature mapping

**User update:** Pleiades and Beehive datasets downloaded and loaded as DataFrames. Many nulls found; kept only ra, dec, parallax, pmra, pmdec.

**Second-guessing notes**
1. **Keep source_id** (essential for cross-matching the two methods and for comparing with CG18/Gao19 member lists).
2. **Keep G and bp_rp (or phot_bp/rp_mean_mag)** even though they are not clustering features. They are needed for CMD validation (Phase 5; AG23 Fig. 5, Deb22 Fig. 3, Gao19 Figs. 2-3). Nulls are expected in radial_velocity columns, so do not let them drive row-dropping. Drop rows null only in the 5 astrometric columns plus source_id/photometry you need.
3. **Report how many rows the null-drop removed** and whether they were faint stars (check G distribution before and after).
4. **Beehive is a different regime than Pleiades:** AG23 centre for Praesepe is about (-36.1, -12.9) mas/yr (vs Pleiades (19.9, -45.5)); distance about 185 pc vs 136 pc.

**Plots for Phase 2 and why**
| Plot | Insight | Where in literature |
|---|---|---|
| PM plane, hexbin or log-density (plus scatter zoom) | Cluster is a compact clump over a broad field; shows if field density swamps it | AG23 Fig. 1; Deb22 Fig. 2; Gao19 Figs. 2-3 |
| pmra and pmdec histograms, log y-axis, with Gaussian fit | Peak position and width -> initial guesses for MLE; defines 3 sigma window | AG23 Sec. 2; Deb22 Fig. 2 |
| Parallax histogram | Cluster peak vs field; confirm distance (Pleiades about 7.35 mas, Praesepe about 5.4 mas) | Deb22 Fig. 2; Gao19 Fig. 5 |
| Sky scatter (ra vs dec), selected vs all | Spatial concentration; edge effects of the cone | CG18 Fig. 8; Deb22 Figs. 3, 4 |
| Parallax vs angular distance from centre | Cluster members cluster at near-constant parallax; field scatter | Gao19 Figs. 4, 8 |
| Parallax vs pmra/pmdec | Depth effect/correlation; tests the circular-cluster assumption | Gao19 Fig. 6, Eqs. 7-8 |
| 7th-NN distance histogram (preview) | Shows bimodality of cluster vs field; informs DBSCAN eps | AG23 Fig. 2; Deb22 Fig. 2 |
| Pairplot / correlation matrix of 5 features | Which features are correlated; why standardisation matters | CG18 Fig. 2 (correlation of astrometric parameters) |

**Closest literature match to "Phase 2":** AG23 Sec. 2 (data + 3 sigma region in VPD, Fig. 1) and Deb22 Sec. 3.1 with Fig. 2 (small-radius pass: d_NN histogram, PM plane, CMD, and three histograms of pi, pmra, pmdec).

**Open questions**
1. Did you also keep photometry/source_id? If not, re-select from the original CSV.
2. How many rows before/after dropping nulls for each cluster?
3. Still waiting on Castro-Ginard 2018.

**Next step:** Produce plots 1-3 (PM plane, PM histograms, parallax histogram) for Pleiades and report what the peak positions are compared with AG23 Table 1 values.
