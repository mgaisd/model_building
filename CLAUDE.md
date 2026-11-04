# CLAUDE.md

Context for Claude Code sessions in this repo. Detailed explanations and the study plan live in [notes/](./notes).

## Project goal (Marcel's task)

Simplified dark-shower (hidden valley, FCDC) model development: fix Nc, Nf, Λ and probVector, and test by comparing observables whether the complete model's behaviour is covered by the simplified model within uncertainties or via reweighting. Target: only **mπ** and **rinv** free in the dark sector, with Λ set by a binary choice of mπ/Λ (2-body vs 3-body regime), plus mediator mass and couplings.

**Not in scope here:** Nc/Nf variations. A colleague already studies those (effective vs simplified observables; Nc matters more), now rerunning with newer jet collections (e.g. dark partons before hadronization). Don't duplicate that work.

## Key physics

- **Complete model** (`spectrum = 'fcdc'`, `fcdcSpectrum` in `svjHelper.py`): Nf separate dark flavours, the first Ns "stable". Off-diagonal pions containing a stable quark are invisible; all others decay to SM quarks. rinv follows from flavour counting, rinv = N_stable/(Nf²−1) (`fcdc_rinv`); Nc = Nf = 3, Ns = 1 gives rinv = 0.5.
- **Simplified model** (`spectrum = 'fcdcSimp'`, `fcdcSimpSpectrum`): one generic dark-quark species; each pion goes invisible with BR = rinv (free input). Comparing both at equal Λ, mπ/Λ, probVector and Nc = Nf = 3 isolates flavour-correlated vs independent per-hadron invisibility.
- **Λ (`scale`)**: dark confinement scale. Drives the shower (`HiddenValley:setLambda = on`, running α_D) and the whole spectrum via `masses_snowmass`: mπ = r·Λ, mq ≈ Λ + (mπ/5.5)²/Λ, mρ = Λ·sqrt(5.76 + 1.5 r²), with r = mπ/Λ.
- **2-body vs 3-body**: ρ → ππ for r < ~1.52, ρ → π q q̄ (or q q̄) above. Repo uses **r = 0.6** (2-body) and **r = 1.7** (3-body), i.e. Λ = mπ/0.6 or mπ/1.7.
- **probVector (`pvector`)**: probability string fragmentation makes a vector (ρ) instead of a pseudoscalar (π). Pythia default 0.75 (spin counting); repo uses 0.333 to mimic SM QCD (`configs/model_fcdc_base.py`). In 2-body it changes only multiplicity/substructure; in 3-body it shifts the effective rinv (`fcdc_rinv_3body`, `fcdc_rinv_3body_simp`), e.g. input 0.5 → 0.379 (complete) / 0.395 (simplified) at r = 1.7, pvector 0.333; 0.295 / 0.324 at pvector 0.75.

## Key files

- `svjHelper.py`: model spectra, `masses_snowmass`, rinv formulas, config generators (`fcdc_config`, `fcdc_config_simp`, `fcdc_configs_NcNf1`).
- `configs/model_fcdc_base.py`: shared base (mmed 1000, gq 0.25, pvector 0.333, ...). `model_fcdc_mpi6[_3body]`, `model_fcdc_mpi10[_3body]`, `model_fcdc_10[_3body]`: complete model; `model_snowmass_fcdclike*`: simplified counterparts.
- `configs/model_fcdc_lambdascan.py`: Λ scan at mπ = 6, r ∈ {0.3, 0.6, 1.0, 1.4, 1.6, 1.7, 2.0, 2.5}, each as a `<tag>_full` / `<tag>_simp` pair (e.g. `r0p6_full`). Cards checked, no events generated yet.
- `run_model`, `run_fcdc_models.py`: generation (cards → Pythia → Delphes → histograms). `Histogram.py`: histogram filling. `Plots.py`, `Metaplot.py`, `NcPlots.py`: plotting from `Hists.pkl` / `Meta.pkl`.
- Pythia is pinned at 8.310; uncertainty weights (`setabsigma`) need ≥ 8.313. Settle the version with the task force before final samples.

## Current plan

Start with steps 2-4 of [notes/lambda-probvector-plan.md](./notes/lambda-probvector-plan.md): Λ scan within each regime, probVector scan (0.333/0.5/0.75 at r = 0.6 and 1.7), complete vs simplified. Exact sample matrix, run commands and plot list: [notes/steps2-4-samples-and-plots.md](./notes/steps2-4-samples-and-plots.md). Samples for r = 0.6 and 1.7 likely already exist in Emily's EOS area (`/eos/uscms/store/user/easmith/svj/models/fcdc/`).
