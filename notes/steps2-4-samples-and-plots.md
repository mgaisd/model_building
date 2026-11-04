# Steps 2 to 4: which samples to generate and which plots to make

All numbers below were computed with the repo's own functions (`svjHelper.py`). The scan config was checked by generating its Pythia cards (`--steps cards`) in a scratch environment, but no events were generated.

---

## A. Background: complete vs. simplified model

Both models use the same production: an s-channel Z′ (mmed = 1000, gq = 0.25, gχ = 1/√(Nc·Nf)) decaying to dark quarks, which shower and hadronize in Pythia's Hidden Valley module. Λ, Nc and Nf set the shower and probVector sets the fragmentation in both. **The difference is how a dark hadron "decides" to be invisible.**

### Complete model (`spectrum = 'fcdc'`, `fcdcSpectrum`, `svjHelper.py:527`)
- Nf separate dark-quark flavours (`separateFlav = on`). The first **Ns** flavours are "stable": they have no flavour-changing coupling back to the SM.
- Every meson is tracked by its flavour content, giving π_ij and ρ_ij for i,j = 1..Nf.
  - **Diagonal pions** (i = j) decay to SM quarks (mass insertion, so heavier quarks are preferred).
  - **Off-diagonal pions** that contain a stable quark are **stable**, so they are invisible (dark matter).
  - All other off-diagonal pions decay to SM quarks.
- Invisibility is therefore not a free parameter. It follows from flavour counting: rinv = N_stable / (Nf² − 1) (`fcdc_rinv`), with the η′ dropped. For Nc = Nf = 3 there is only Ns = 1, which gives **rinv = 0.5**.
- Inputs: Nc, Nf, Ns, Λ, mπ/Λ, probVector.

### Simplified model (`spectrum = 'fcdcSimp'`, `fcdcSimpSpectrum`, `svjHelper.py:567`)
- Nc = Nf = 3 are fixed and there is a single generic dark-quark species, so there is no per-flavour bookkeeping.
- There is one generic diagonal pion (4900111) and one off-diagonal pion (4900211). **Each pion decays invisibly with branching ratio = rinv**, and otherwise decays to SM quarks.
- Invisibility is a coin flip per hadron with probability rinv, and **rinv is the free input parameter.**
- Inputs: rinv, mπ, regime (mπ/Λ), with Nc, Nf and probVector fixed.

### What the comparison actually tests
At the same Λ, mπ/Λ and probVector, and with Nc = Nf = 3 on both sides, the shower and fragmentation are identical. Any difference in observables then comes from replacing **flavour-correlated** invisibility with an **independent per-hadron** branching ratio. Flavour is conserved along the string, so in the complete model the stable mesons are correlated within a jet. The simplified model loses that correlation, and this shows up in the jet-by-jet rinv spread and in substructure. Your colleague's study varies Nc and Nf on the complete side. Yours varies Λ and probVector on both sides.

## B. Background: 2-body vs. 3-body regime and how the parameters are set

Masses come from `masses_snowmass(config, scale=Λ, mpi_over_scale=r)` (lattice-inspired fits from arXiv:2203.09503):
- mπ = r·Λ
- constituent quark mass mq = Λ + (mπ/5.5)²/Λ
- mρ = Λ·sqrt(5.76 + 1.5 r²)

mρ grows more slowly than mπ as r increases, so at r ≈ 1.52 the ρ drops below the ππ threshold:

| | 2-body (r < 1.52) | 3-body (r > 1.52) |
|---|---|---|
| repo choice | r = 0.6 (Λ = 10, mπ = 6, mρ = 25.1) | r = 1.7 (Λ = 3.53, mπ = 6, mρ = 11.2) |
| complete model, ρ decays | every ρ → ππ (`darkRho2BodyDecay`, equal BRs to the allowed flavour pairs) | "protected" ρ (containing a stable quark) → π_stable + q q̄ (3-body, `darkRho3BodyDecay`). Other ρ → q q̄ directly (`democraticDecay`) |
| simplified model, ρ decays | ρ → ππ, and those pions then decay with rinv | ρ → π_stable q q̄ with BR = rinv, else ρ → q q̄ |
| does probVector change rinv? | no (every ρ ends as pions) | yes: ρ's carry only a fraction α of their energy to the invisible pion |
| config files | `model_fcdc_10`, `model_fcdc_mpi6`, `model_snowmass_fcdclike_mpi6` | `model_fcdc_mpi6_3body`, `model_snowmass_fcdclike_mpi6_3body` |

In the 3-body regime the energy-weighted invisible fraction is lower than the input rinv. The repo stores it as `rinvpred_3body` (complete) and `rinv_3body` (simplified) in `Meta.pkl`. Values at mπ = 6 with input rinv = 0.5:

| mπ/Λ | probVector | complete `rinvpred_3body` | simplified `rinv_3body` |
|---|---|---|---|
| 1.6 | 0.333 | 0.373 | 0.390 |
| 1.7 | 0.333 | 0.379 | 0.395 |
| 1.7 | 0.5 | 0.339 | 0.362 |
| 1.7 | 0.75 | 0.295 | 0.324 |
| 2.0 | 0.333 | 0.391 | 0.408 |
| 2.5 | 0.333 | 0.406 | 0.423 |

Two things follow from this table. First, even at identical settings the two models differ by about 0.02 in predicted 3-body rinv (the simplified model treats all ρ alike). Second, probVector moves the 3-body rinv by about 0.08 between 0.333 and 0.75, which is more than the whole 1.6 to 2.5 Λ range moves it.

---

## C. Sample matrix

Fixed throughout: s-channel, mmed = 1000, gq = 0.25, Nc = Nf = 3, mπ = 6, complete with Ns = 1 and simplified with rinv = 0.5. Each point is one complete sample plus one simplified sample.

**Step 2 (Λ scan), probVector = 0.333**

| regime | mπ/Λ | Λ | mq | mρ | status |
|---|---|---|---|---|---|
| 2-body | 0.3 | 20 | 20.06 | 48.56 | new |
| 2-body | 0.6 | 10 | 10.12 | 25.10 | probably exists (Emily's EOS area, same dir name) |
| 2-body | 1.0 | 6 | 6.20 | 16.17 | new |
| 2-body | 1.4 | 4.29 | 4.56 | 12.64 | new (close to threshold) |
| 3-body | 1.6 | 3.75 | 4.07 | 11.62 | new (close to threshold) |
| 3-body | 1.7 | 3.53 | 3.87 | 11.21 | probably exists |
| 3-body | 2.0 | 3 | 3.40 | 10.29 | new |
| 3-body | 2.5 | 2.4 | 2.90 | 9.34 | new |

This is 16 samples, of which 4 probably exist already. The output directory names from the config below match those used in `Metaplot.py` and `NcPlots.py` character for character, so check `/eos/uscms/store/user/easmith/svj/models/fcdc/` before regenerating.

**Step 3 (probVector), at r = 0.6 and r = 1.7:** pvector = 0.5 and 0.75 (0.333 is already in step 2). That gives 2 regimes × 2 values × (complete + simplified) = **8 samples**.

**Step 4 (complete vs. simplified)** needs no extra samples: every point above already comes as a complete/simplified pair.

**Statistics:** 10k events per sample (as in the Nc study, `plots_AllNcNf_10k`). Run 1k first to check that everything works end to end.

**Optional later:** repeat the r endpoints (0.3, 1.4, 1.6, 2.5) at mπ = 20 to check that the conclusion does not depend on mπ.

## D. How to generate

The config [configs/model_fcdc_lambdascan.py](../configs/model_fcdc_lambdascan.py) builds all 16 step-2 configs (`r0p3_full`, `r0p3_simp`, …). It is in `configs/`.

```bash
source init.sh
# step 2: all 16 samples
python run_fcdc_models.py --config-name configs/model_fcdc_lambdascan.py \
    --args "--steps all --events 10000" --outdir models/fcdc/ --nproc 8
# step 3: probVector override from the command line (verified: lands in a pvector-0.75 directory)
for pv in 0.5 0.75; do
  python run_fcdc_models.py --config-name configs/model_fcdc_lambdascan.py \
      --args "--steps all --events 10000 --pvector $pv" --suffix pv$pv --outdir models/fcdc/ --nproc 8
done
```
The step-3 loop runs every r. To limit it to r = 0.6 and 1.7, either set `ratios = [0.6, 1.7]` in a copy of the config, or call `./run_model helper -C configs/model_fcdc_lambdascan.py -O config.r0p6_full --pvector 0.75 --steps all --events 10000 --dir models/fcdc` for each of the 4 objects.

## E. Plots

All of these reuse `Hists.pkl` and `Meta.pkl` from the existing pipeline. Only the sample lists in the scripts change.

1. **Λ scan, complete model only, one regime per plot** (step 2). Use the `Plots.py` layout with a sample list of the 4 complete samples in a regime, normalized overlays, and a ratio panel to the repo default (0.6 or 1.7). Do the same for the simplified model.
   - Look at the shower first: `DPJet12_pt` (dark-parton jets) and `n_pion` / `n_rho`, to see how much more dark radiation a larger Λ gives.
   - Then the analysis-level observables: `MET`, `MT`, `DeltaPhi_MET_Jet12`, `Jet12_girth`, `Jet12_ptD`, `Jet12_majoraxis`, `Jet12_sdmass`.
   - Then invisibility: `stable_invisible_fraction`, `DHIVJet12_rinv_proj`.
2. **Summary vs. mπ/Λ** (step 2). Use `Metaplot.py --types stat -x scale` (or add mπ/Λ to the metadata) with the mean ± stderr of the rinv quantities. Put both regimes on one plot with a vertical line at 1.52. If the quantities are flat inside each regime and jump at the threshold, the binary choice is justified.
3. **probVector overlays** (step 3). Show three curves per regime per model. Expect 2-body to change mainly `n_pion` / `n_rho` and substructure, and 3-body to change `MET`, `MT` and the rinv quantities.
4. **Complete vs. simplified** (step 4). Overlay each pair with a ratio panel at each r. Plot `DHIVJet12_rinv_proj` per jet in particular: its width shows the lost flavour correlation even when the means agree.
5. **3-body rinv check.** Compare the generated `stable_invisible_fraction` against `rinvpred_3body` / `rinv_3body` for every 3-body point and probVector. If they agree, the formula can be used to translate between probVector values instead of generating new samples.

For a first quantitative number per observable, put the χ²/ndf between the two normalized histograms (stat errors only) in the ratio-panel legend. This is enough to rank which observables differ before building the full metric from step 1 of the main plan.
