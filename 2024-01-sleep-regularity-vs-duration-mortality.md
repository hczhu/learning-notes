- **Source**: Windred DP, Burns AC, Lane JM, Saxena R, Rutter MK, Cain SW, Phillips AJK. "Sleep regularity is a stronger predictor of mortality risk than sleep duration: A prospective cohort study." *SLEEP* 47(1), 2024 (advance access 21 Sep 2023). Open access. DOI: [10.1093/sleep/zsad253](https://doi.org/10.1093/sleep/zsad253)
- **One-liner**: In >60,000 UK Biobank adults with a week of wrist accelerometry, **how regular your sleep timing is** predicts all-cause, cardiometabolic, and cancer mortality more strongly than **how long you sleep** — and regularity is a more direct proxy for circadian health.

- ## Key Message
	- Sleep guidelines have long optimized **duration** (the 7–9 h target). This study argues the field has been focusing on the wrong metric: **day-to-day consistency of sleep–wake timing** is a stronger, more robust mortality predictor.
	- Practical takeaway from the authors: **fall asleep and wake up within ~1-hour windows each day**. The most regular sleepers (top 20%) fell asleep/woke within ~1-hour windows; the least regular (bottom 20%) spanned ~3-hour windows.

- ## Study Design
	- **Cohort**: UK Biobank, N = **60,977** participants with valid sleep-regularity scores (from 103,669 who wore devices).
	- **Ages**: mean 62.8 ± 7.8 years (recruited 40–69); **55.0% female**, 97.2% white.
	- **Exposure measurement**: Axivity AX3 tri-axial accelerometer on the dominant wrist, worn **7 days** under free-living conditions (2013–2016), logged at 100 Hz. Over **10 million hours** of actigraphy total.
	- **Follow-up**: mean 6.30 ± 0.83 years (up to 7.8 years); mortality from NHS Digital / NHS Central Register, ICD-10 coded, through March 2021.
	- **Deaths**: **1859 all-cause** (4.84 per 1000 person-years) — 1092 cancer, 377 cardiometabolic, remainder other-cause.

- ## The Two Metrics
	- **Sleep Regularity Index (SRI)**: probability that sleep–wake state matches at two time points exactly 24 h apart, averaged over the recording. **100 = perfectly regular**, **0 = random**. Computed via the `GGIR` + `sleepreg` R packages (accounts for naps, fragmented sleep, non-wear). Cohort median SRI = **81.0** (IQR 73.8–86.3); distribution negatively skewed.
	- **Sleep duration**: daily sustained inactivity between sleep onset and offset, averaged per person. Mean **6.77 ± 1.00 h**.
	- Both were split into **quintiles**; the top four quintiles were each compared against the lowest (reference) quintile.
	- **Statistical models**: Cox proportional hazards (all-cause) and competing-risks sub-hazards (cause-specific). Three model sets — (1) SRI only, (2) duration only, (3) both — each run "minimally adjusted" (age, sex, ethnicity) and "fully adjusted" (+ physical activity, employment, income, deprivation, social visits, smoking, urbanicity, shift work, cholesterol/hypertension medication).

- ## Headline Results
	- ### Sleep regularity → mortality (monotonic: more regular = lower risk)
		- The top four SRI quintiles had a **20–48% lower all-cause mortality** risk vs. the least-regular quintile (fully adjusted p < .001 to .004).
		- Cause-specific reductions (top four vs. bottom quintile): **cancer 16–39% lower** (p < .001 to .017); **cardiometabolic 22–57% lower** (p < .001 to .048).
		- Best-case all-cause hazard ratios (80–100th percentile): minimal model **HR = 0.52** [0.45–0.60]; fully adjusted **HR = 0.70** [0.59–0.83].
		- Largest cause-specific effects (minimal / full): cardiometabolic **HR 0.43 / 0.62**; cancer **HR 0.61 / 0.76**; other-cause **HR 0.39 / 0.60**.
	- ### Sleep duration → mortality (non-linear, U-shaped)
		- Duration showed a **U-shaped** relationship with all-cause mortality in the minimal model; in the fully adjusted model longer duration up to the 80–100th percentile trended to lower risk (HR = 0.76 [0.65–0.89]).
		- Duration was a **significant predictor of cardiometabolic and other-cause** mortality, but **not a significant predictor of cancer** mortality.
	- ### Head-to-head: regularity wins
		- In combined models (both predictors), **SRI stayed a significant mortality predictor**, and minimum hazard ratios were consistently lower for SRI models than duration models.
		- Formal model comparison (Akaike Information Criterion): SRI-only models fit all-cause mortality **better** than duration-only models (minimal p < .001; full p = .005).
		- Nested likelihood-ratio tests: adding duration to an SRI model did **not** explain significant additional variance (p = .14–.20) — i.e., duration adds little once regularity is known, but not vice versa.
		- After adjusting for duration, SRI remained a significant predictor of **all-cause, cancer, and other-cause** mortality (but **not cardiometabolic** in the fully adjusted combined model).

- ## Notable Nuances
	- **Regularity ≠ duration**: they are only weakly related. Longer duration associated with higher SRI up to ~7.83 h, beyond which longer sleep associated with *lower* SRI (linear fit R² only 0.085) — so SRI captures something duration does not.
	- **Cancer**: irregular sleep predicted higher cancer mortality, whereas short duration did not. The SRI–cancer link held even in people **without** preexisting cancer, supporting a circadian-disruption mechanism (irregular light exposure, clock disruption promoting tumor progression).
	- **Long sleep caveat**: the longest-sleeping quintile here was >7.56 h — not the extreme >9 h cutoff used in studies that find long-sleep harm — so no elevated risk was seen at the top quintile.
	- **Mechanism framing**: SRI is argued to be a more *direct* proxy for **circadian disruption** than duration; irregular sleep means irregular light, meals, and activity, disrupting central and peripheral clocks.

- ## Limitations
	- Only a **single week** of accelerometry — a snapshot, not longitudinal sleep behavior.
	- Covariates were collected at baseline (2006–2010), **years before** the accelerometer week; some are not temporally stable.
	- Cohort is **older, mostly white, healthy-volunteer** (UK Biobank) — generalizability across ages/ethnicities/cultures is untested.
	- **Observational**: sleep regularity may be both a *cause and a marker* of premature mortality risk; causality not established. Fully adjusted models may over-adjust (some covariates are mediators, e.g. smoking).

- ## Why It Matters
	- Challenges the assumption that **duration is the primary sleep-health target**; positions **regularity** as an equally or more important, and possibly **easier-to-modify**, intervention target (extending sleep is hard; regularizing timing may be more feasible).
