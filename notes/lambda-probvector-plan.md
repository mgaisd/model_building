# Λ and probVector: what they are, and a study plan

Scope: Marcel's part of the simplified-model task (fix Nc, Nf, Λ, probVector; test coverage of the complete model). Nc/Nf variations are already covered by a colleague (incl. the newer jet collections such as dark partons before hadronization), so they are not repeated here.

## 1. What the two parameters are

### Λ (dark confinement scale, `scale` in the configs)
Physically: the scale where the dark coupling becomes strong and dark quarks bind into dark hadrons, i.e. the dark analogue of Λ_QCD.

In the repo it enters in two places at once:

1. **Shower strength.** `HiddenValley:setLambda = on`, `alphaOrder = 1`, `Lambda = scale` (`svjHelper.py:796-805`). Pythia then runs α_D(Q) = 1/(b0 ln(Q²/Λ²)) with b0 depending on Nc and Nf. At fixed mediator mass, a larger Λ means a larger α_D at the hard scale, so more dark radiation, more dark hadrons, wider jets. (Pythia also ties the dark-shower cutoff to Λ; inferred from the Pythia HV docs, worth confirming in a generated card log.)
2. **The whole mass spectrum** via `masses_snowmass` (`svjHelper.py:26`, from arXiv:2203.09503):
   - mπ = (mπ/Λ) · Λ
   - constituent dark quark mass mq ≈ Λ + (mπ/5.5)²/Λ (this is what Pythia uses for string breaking)
   - mρ = Λ · sqrt(5.76 + 1.5 (mπ/Λ)²)

Because mρ is tied to Λ, the ratio mπ/Λ decides how the ρ decays:

| mπ/Λ | mρ/Λ | mρ/mπ | ρ decay |
|---|---|---|---|
| 0.3 | 2.43 | 8.1 | 2-body ρ → ππ |
| 0.6 (repo default) | 2.51 | 4.2 | 2-body |
| 1.0 | 2.69 | 2.7 | 2-body |
| 1.4 | 2.95 | 2.1 | 2-body |
| ~1.52 | | 2.0 | threshold |
| 1.7 (repo 3-body) | 3.18 | 1.87 | 3-body ρ → π q q̄ (protected ρ) or ρ → q q̄ |
| 2.5 | 3.89 | 1.56 | 3-body |

So in the simplified model "Λ from a binary choice" means: pick mπ (free), pick the regime, and Λ = mπ/0.6 or mπ/1.7. The open question is whether one ratio per regime is representative of the whole regime.

Existing configs: `model_fcdc_10` (Λ=10, mπ=6, 2-body), `model_fcdc_10_3body` (Λ=10, mπ=17), `model_fcdc_mpi6[_3body]` and `model_fcdc_mpi10[_3body]` (fixed mπ, both regimes), plus the matching `model_snowmass_fcdclike*` simplified versions.

### probVector (`pvector`)
Physically: in Pythia's HV string fragmentation, the probability that a newly formed dark meson is a vector (ρ) rather than a pseudoscalar (π), i.e. V/(V+PS). Naive spin counting gives 0.75 (Pythia's default). The repo uses 0.333, chosen to mirror SM QCD (Monash/CP5 `mesonUDvector = 0.5` is V/PS, so V/(V+PS) = 1/3; comment in `configs/model_fcdc_base.py:7-9`). It is one number for all flavours.

What it does depends on the regime:
- **2-body (ρ → ππ):** every ρ becomes two pions, so the predicted rinv (`fcdc_rinv`) does not depend on probVector. It changes the hadron multiplicity and momentum sharing (substructure), not the invisible fraction.
- **3-body:** probVector enters rinv directly. In `fcdc_rinv_3body` (`svjHelper.py:142`) the pion and ρ contributions are weighted by (1−pvector) and pvector, with κ = mρ/mπ and α = ⟨E_π⟩/⟨E_ρ⟩ from 3-body phase space. The simplified model (`fcdcSimp`, ρ → π_stable q q̄ with BR = rinv) has the same dependence through `fcdc_rinv_3body_simp`, so the "effective" rinv a user gets for a given input rinv depends on probVector.

## 2. Ordered plan

**Step 0. Coordinate (before generating anything).**
- With the colleague doing Nc/Nf (and Emily, who is listed on the same task in the task-force notes): agree on one common setup so results combine: Pythia version, mmed and couplings, jet collections, Histogram.py version, statistics, and the Nc=Nf=3 complete-model samples they already have (reuse them as your baseline).
- With Kevin (Lund string and shower uncertainties): his variation bands are the natural yardstick for "covered by uncertainty". Ask for the variations and their size.
- Note: the repo pins Pythia 8.310, while the task-force notes list `setabsigma` (needed for uncertainties) in 8.313 and fragmentation bug fixes in 8.314 and 8.318. Decide the version together before producing final samples.

**Step 1. Make the comparison quantitative.** Right now comparisons are by-eye overlays (`Metaplot.py`, `NcPlots.py`), with no reweighting or coverage test. Add one metric reused by every step below, e.g. per-bin ratio plus a χ²/KS-type distance, and a "difference / uncertainty band" figure of merit per observable and per jet collection. This is shared infrastructure, so build it together with the Nc/Nf colleague rather than twice.

**Step 2. Λ within each regime (the core of your part).** Complete fcdc model at Nc=Nf=3, fixed mπ (start with mπ=6 and 10, already configured), fixed mmed:
- 2-body scan: mπ/Λ = 0.3, 0.6, 1.0, 1.4
- 3-body scan: mπ/Λ = 1.6, 1.7, 2.0, 2.5
- Question: inside one regime, do observables stay within the yardstick from Step 1? If yes, one ratio per regime is justified; if not, find which ratio is most central and how big the residual spread is (that becomes an uncertainty).
- Watch near the threshold (~1.52), and check the dark-parton and dark-hadron jet collections to separate the shower effect (α_D) from the spectrum/decay effect.
- Repeat at one or two larger mπ to see whether the conclusion is mπ-independent.

**Step 3. probVector.** Scan pvector = 0.333, 0.5, 0.75 in both regimes (at the Λ chosen in Step 2).
- 2-body: expect only substructure/multiplicity changes; test whether they sit inside the uncertainty.
- 3-body: expect rinv-level changes. Test whether a simplified sample with rinv set to the matching effective value reproduces the complete model at another probVector. If it does, probVector is absorbed into the rinv definition and can stay fixed at 0.333.

**Step 4. Complete vs. simplified at the chosen values.** With Nc=Nf=3, the chosen ratio per regime and probVector fixed, compare `fcdc` against `fcdcSimp` across the (Nf, Ns) → rinv grid already produced by `fcdc_configs_NcNf1(simp=True)`. This is the direct test of "only mπ and rinv free" and uses the same metric.

**Step 5. Reweighting vs. uncertainty.** For each residual difference, decide which treatment fits:
- Decay-level choices (rinv, and probVector in the 3-body regime) can plausibly be reweighted event by event from counts of invisible/visible decays or ρ/π multiplicities (inferred; needs a closure test against a generated sample).
- Shower-level effects (Λ through α_D) are not easily reweighted in the HV shower; these likely become an uncertainty, combined with Kevin's variations.

**Step 6. Hand-off to the Nc/Nf result (only after their study).** One open question sits at the boundary of both studies: whether a different Λ can absorb part of the Nc dependence (both change α_D at the hard scale). Raise it with the colleague rather than running it independently.

**Step 7. Write up as a config.** The task force deliverable is the framework, not plots: encode the result as a simplified-model config generator taking (mπ, rinv, regime, mmed, couplings) with the fixed Nc, Nf, Λ ratio and probVector, plus the uncertainty or reweighting prescription.
