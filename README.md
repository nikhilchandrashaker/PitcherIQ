# PitcherIQ: Findings Report

All numbers come from `reports/*.csv` and `reports/run_log.txt`. Everything here is **association, not causation**.

## 0. What the data can and cannot support
- **No game-by-game MLB file and no dated surgery list were supplied.** So "TJS next season" forecasting and acute:chronic workload ratios across MLB could not be built. Substitutes: season-level workload change (all MLB, 2015–23) and game-level workload for 18 Japanese-born pitchers (2,159 regular-season appearances).
- **2020 is missing** in the season files; only consecutive-season pairs are used.
- `TJS_final_v3` is pitcher-level, ~49% positive (a case-control-style sample), so any probability from it is not a population injury risk.
- Kinematics (`baseball_data.csv`) is 25 youth pitchers (ages 12–18), with an anonymous ID that does not link to MLB data.
- The `teams` column in `mlbpitching.csv` is unreliable pre-1960; everything is normalised by team-games.

## 1. 150 years of pitching (Fig 01–02, `historical_era_summary.csv`)
Complete-game share of team-games fell from ~92% (1871–92) to ~2% (2010–21). Strikeouts per 9 rose from 2.7 to 8.1. Pitchers used per 162 team-games rose from ~9 to ~28. CG rate and staff size correlate at r = −0.70 across years. Innings per team-game stayed ~8.9 throughout: the same innings are now spread over many more arms.

## 2. What makes a pitcher effective (Fig 03)
Correlation with xwOBA allowed (PA ≥ 100): K% −0.67, whiff% −0.54, fastball velocity −0.29, hard-hit% +0.49, BB% +0.24. Average fastball velocity rose from 92.8 to 94.1 mph (2015–23).

## 3. Tommy John pitch profiles (Fig 04, `tjs_feature_comparison.csv`)
TJS pitchers (n=77) vs controls (n=79): higher max fastball velocity (97.9 vs 96.7 mph, p=.008), higher K% and whiff%, more innings per game. But the strongest difference is **seasons observed** (4.8 vs 3.4, d=0.77): surgery pitchers are longer-tenured. Adjusted logit: seasons observed OR-scale coef 0.74 (p<.001), IP/game 0.50 (p=.008), max velocity 0.37 (p=.059, no longer significant). Velocity is not clearly independent of exposure/role.

## 4. Workload (Fig 05–07)
- **Season level:** pitchers who *increased* workload had slightly *better* next-year velocity change (+0.13 mph per log-unit of workload ratio, p=.002, controlling for age, prior velocity, year) and lower xwOBA. This is selection: healthy, effective pitchers get more work, and declining ones get cut. It is not evidence that more work helps.
- **Game level (Japanese-born pitchers):** workload spikes (>+30% vs prior 3 outings) show no next-start velocity effect (coef 0.049, p=.56; within-pitcher, clustered SE). Small sample (18 pitchers). Absence of a detectable effect here is not evidence of safety.

## 5. Career archetypes (Fig 08, `war_cluster_profiles.csv`)
Top-500 WAR, k=3 (silhouette 0.31, bootstrap ARI 0.75 mean / 0.24 min). Cluster separation is **moderate**, closer to a continuum than discrete types: (0) n=275, solid mid-WAR careers, 1% HOF; (1) n=67, short high-efficiency careers (ERA+ 130, WAR/162 5.0), 19% HOF; (2) n=158, long high-volume careers (3,586 IP, WAR 61), 37% HOF. The sample is conditioned on being top-500 by WAR, so it cannot say anything about careers that ended early.

## 6. Kinematics (Fig 09)
Youth ball speed is driven mainly by age and body size (height r=0.84). Trunk angular velocity adds a small effect in a mixed model (p=.044); pelvis none. Leave-pitcher-out CV R²≈0.60 (Ridge) vs −0.09 baseline: the signal is mostly growth, not technique.

## 7. Model 1: injury classification (Fig 10, `injury_model_cv.csv`)
10×5 repeated CV, best LogReg: pitch-profile only AUC 0.64 (p=.02 by label permutation); pitch + exposure 0.76; **exposure only 0.78**. Pitch traits carry weak signal; most apparent signal is tenure/usage, which likely reflects how the sample was built. Not a forecast, and not medical risk.

## 8. Model 2: next-season performance (Fig 11, `performance_model_results.csv`)
Temporal split: train targets 2016–18, validate 2019, test 2022–23 (n=722). Next-season xwOBA RMSE: Ridge+workload 0.0298, vs repeating last year 0.0389 and league mean 0.0345 (R² 0.25). Gain vs persistence 0.009 (95% bootstrap CI 0.0075–0.0107). Workload-change features add only ~0.0002. Tree models did not beat Ridge. K−BB% R² ≈ 0.31.

## 9. Model 3: longevity (Fig 12, `longevity_cox.csv`)
Cohort: 586 pitchers first seen 2016–19, 377 "exits" (no later appearance), 209 censored. Cox hazard ratios (per SD): velocity 0.86, K−BB% 0.82, log IP 0.81, hard-hit% 1.18, age 1.34; starter-like 0.68. 5-fold concordance 0.71 ± 0.03. K−BB% violates proportional hazards (p<.005), so its effect varies over time. "Exit" includes demotion and falling under the leaderboard threshold, not only retirement; first appearance is a debut proxy.

## 10. Reproducibility and next steps
Run `src/run_all.py`. To make the injury work forecasting-grade you need dated surgeries (e.g., a TJS tracker) plus game logs with pitch counts; the pipeline would then plug in at `01_data_audit_and_master.py`.
