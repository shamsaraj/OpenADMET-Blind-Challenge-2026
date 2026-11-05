# CYP3A4 structure prediction — protein–ligand pose

**Submission:** `24_11_frozen_structures.zip`
**Final score:** LDDT-PLI **0.3608**, BiSyRMSD 4.4045 Å, LDDT-LP 0.8308, Coverage 1.0 — rank
**34 / 38**, Tier 2 of 4.
**Author:** Jamal Shamsara

---

## 1. Task

Predict the 3D structure of the CYP3A4 complex for 20 compounds **from SMILES alone**, scored
by **LDDT-PLI** against blinded Cryo-EM structures, with BiSyRMSD, LDDT-LP and Coverage
reported alongside.

Two hard gates apply: a complex that fails the organisers' structure validator incurs
LDDT-PLI 0 plus a 20 Å penalty, and a complex that fails **PoseBusters** scores LDDT-PLI 0.
Both are checked locally before every submission, and both are treated as part of the method
rather than as a formality — an earlier internal build lost a structure to a PoseBusters
failure that a visual check had passed.

## 2. The 20 ligands

![The 20 challenge ligands](figures/fig4_pose_ligands.png)

*Donor nitrogens — neutral aromatic N carrying no hydrogen, i.e. with a lone pair available to
ligate iron — are highlighted. Fe–N is the distance in the submitted pose.*

The set divides cleanly on a single chemical criterion that determines which binding mode is
even available:

| Group | n | Composition |
|---|---|---|
| **Free aromatic N** (type-II capable) | **12** | pyridines, imidazoles, a benzimidazole, 1,2,3- and 1,2,4-triazoles |
| **No donor nitrogen** | **8** | sulfonamides, amides, a morpholine, a 2-oxindole |

Compound size is tightly distributed: 18–27 heavy atoms, median 24.

Three of the twelve (`OCNT-2395436`, `OCNT-2395464`, `OCNT-2395789`) share an
aminopyridine-nitrile core and required special handling (§4.1). None of the 20 carries a
designed covalent warhead; several are mechanism-based inactivators, which is a different
property and does not imply a covalent pose.

## 3. Receptor preparation

**Ten holo CYP3A4 X-ray entries** were used as docking targets: `3UA1`, `4D6Z`, `6BCZ`,
`6BDH`, `6DA5`, `6DAA`, `6MA6`, `6MA7`, `6OOB`, `7UFB`. They were selected from a survey of
all CYP3A4 PDB entries by filtering on holo + heme present + X-ray + resolution ≤ 2.5 Å +
ligand heavy-atom range, then sorting by resolution and deduplicating by ligand code.

Design units, including heme as a cofactor, were built with **OpenEye Spruce**. Where an entry
contains several crystallographically independent copies, only the copy nearest the ligand is
retained — the entries share residue numbering across copies, so keying on residue number
alone silently merges them.

**Two limitations of this panel were quantified after the fact and are reported here because
they bound what the method could achieve.**

First, the panel is far less diverse than selection on resolution implies. Across the 94
pocket residues present in all ten receptors, pairwise side-chain RMSD is median **0.71 Å**
(range 0.29–1.14), and average-linkage clustering returns **a single cluster even at a 2.0 Å
cut**. Only Phe-304 and Arg-217 move appreciably (≈2.0 Å). Two of the ten (`6DA5`, `6DAA`)
have crystal ligands with **identical SMILES**, and five of the ten are the same
ritonavir-analogue chemotype.

Second, `4D6Z` is a defective target. It was co-crystallised with **both** its drug-like
ligand and **imidazole**, and the imidazole occupies the axial heme site
(Kaur *et al.*, *J. Med. Chem.* 2016, **59**, 4210–4220). Our pipeline selected the drug-like
ligand as the reference, so Spruce stripped the imidazole — leaving an empty axial site in a
protein shaped around an occupant — and centred the docking box 7.33 Å off the iron. `4D6Z`
has the best resolution of all ten (1.93 Å), which is precisely why resolution-based selection
promoted it. It contributes no pose to this submission.

## 4. Ligand preparation

Ligands were prepared from SMILES only; no external 3D geometry entered any pose.

- **Protonation:** OpenEye QUACPAC at pH 7.4, followed by `OEGetReasonableProtomer` for the
  dominant tautomer.
- **Stereochemistry:** unspecified centres enumerated with `OEFlipper`. Two ligands arrive with
  stereochemistry the organisers resolved from the Cryo-EM; those were used as given and never
  re-enumerated.
- **Conformers:** Omega dense sampling.

### 4.1 A protonation correction

`OEGetReasonableProtomer` assigned **+1** to the three aminopyridine-nitriles, protonating the
pyridine ring nitrogen and thereby **removing their only possible iron ligand**. An
independent microspecies calculation showed the **neutral** form is the dominant species at
pH 7.4 for all three, with the next species appearing only above pH 8 — consistent with the
expected pKa, since the amino group's basicity is largely cancelled by the ring nitrile
(pyridine 5.2, 4-aminopyridine 9.1, 3-cyanopyridine 1.45).

All three were re-prepared from the challenge SMILES as given and re-docked. With the correct
species, two of the three reach the 2.0–2.4 Å crystal window (2.26 and 2.25 Å) where as
cations they had no coordinating nitrogen at all. The correction is the reason the type-II
group in §2 numbers 12 and not 9.

## 5. Pose generation

Two independent engines were used on every ligand × receptor pair.

**OpenEye POSIT** (`OEPositMethod_ALL`), a template-guided cascade that tries SHAPEFIT and MCS
overlay, then template-biased HYBRID docking, then template-free FRED, reporting which engine
fired. Ten poses were requested per pair. **On these ligands the cascade falls through**:
SHAPEFIT and MCS fired zero times across the pool and FRED supplied roughly two-thirds of the
poses. The reason is measurable — the highest Tanimoto similarity between any challenge ligand
and any of the ten crystal ligands is **0.497**, and the largest common substructure is a
median of **8 heavy atoms** against ligands of 18–27. The panel offers these compounds
essentially two usable templates, not ten.

**AutoDock Vina 1.2.7** with Meeko, ligands embedded by RDKit ETKDG from SMILES, box
enveloping each receptor's own crystal ligand, `exhaustiveness=24`, 30 poses retained per
pair. Heme is appended as rigid atoms with element-level AutoDock types, since Meeko cannot
template it.

**Vina cannot produce a type-II pose, and this is a property of its scoring function rather
than of the search.** Its closest approach of a free aromatic nitrogen to the iron is **2.79 Å**
over 1,797 poses, and **2.87 Å** over an independent 1,800-pose run — the same wall twice, with
zero poses inside 2.6 Å. Vina has no metal-coordination term, so the iron is treated as an
ordinary atom and its van der Waals repulsion stops a nitrogen at contact distance; a real
dative Fe–N bond at 2.0–2.3 Å lies inside that wall. POSIT's Chemgauss4 does include a metal
term and reaches **2.01 Å**. Vina was therefore used for pose *diversity*, and POSIT for the
coordinating compounds.

## 6. Pose selection

Selection, not generation, is the bottleneck: across the pool the best available pose is far
better than the pose any score selects.

Two selectors were tested head-to-head on the blind set by submitting them as a controlled
A/B over the same pose pool, changing one variable:

| Rule | LDDT-PLI | BiSyRMSD | Coverage |
|---|---|---|---|
| Fe–N coordination for type-II compounds | **0.3939** | 3.945 Å | 1.0 |
| POSIT probability alone | 0.2894 | 5.595 Å | 1.0 |

**Coordination geometry is worth +0.105 LDDT-PLI** — the largest effect measured anywhere in
this work, and the only pose-selection signal validated on blind data. The probability-only
arm placed half of its type-II nitrogens 4–17 Å from the iron, outside the heme site
entirely. `posit_probability` barely discriminates on this set: 62 of 200 candidates share the
value 0.24 and only 3 of 20 compounds have a unique argmax, so "highest probability" resolves
to whatever breaks the tie.

A local control built from cross-docking the ten crystal ligands **failed to predict this**,
calling the two rules indistinguishable (paired Wilcoxon p = 0.58). The reason is now clear:
on crystal ligands POSIT's SHAPEFIT template overlay fires about half the time, while on the
challenge ligands it never fires. The control was measuring a different algorithm, so its
p-values cannot accept or reject a rule for the blind set.

The same cross-docking data does support one weaker, independently validated claim. Receptors
are not interchangeable: median cross-docking RMSD spans 4.70 Å to 6.61 Å. No single best
receptor can be named — bootstrap confidence intervals are ≈2 Å wide — but under
leave-one-ligand-out a better **half** validates, at −0.49 Å (p = 0.037, better on 8 of 10
ligands), with `7UFB`, `6BDH` and `6DAA` in the good group in 9–10 of 10 folds and `4D6Z`,
`6MA7`, `3UA1`, `6OOB` never in it.

### 6.1 The submitted selection

The submitted poses come from a final round in which POSIT was re-run on **7UFB alone** at
**50 poses per compound** for all 20 compounds — 951 poses, correct protonation throughout,
no Vina. Deeper sampling of one good receptor turned out to beat the whole ten-receptor,
ten-pose pool for several compounds: `OCNT-2312554` improved from 3.00 Å to 2.49 Å and
`OCNT-2395436` from 2.76 Å to 2.41 Å, and eleven of twelve donors reached ≤2.49 Å in 7UFB
alone. The earlier pools were **sampling-limited, not receptor-limited** — a result worth
stating because multi-receptor ensemble docking is usually motivated by the opposite
assumption.

Poses were inspected visually in PyMOL. For compounds with a free aromatic nitrogen the
session showed only poses whose nitrogen lay within 4.0 Å of the iron; for the eight with no
donor it showed all 50. One compound, `OCNT-2395712`, reached only 7.59 Å in 7UFB across 48
poses and was re-docked into `6MA6` and `6MA7` — the two receptors whose crystal ligands are
themselves coordinating azoles — where all nine returned poses came from POSIT's
template-biased HYBRID engine, the only place in this work where its template machinery
engaged.

![Coordination geometry of the submitted poses](figures/fig3_pose_feN.png)

| | |
|---|---|
| Receptors used | `7UFB` ×19, `6MA7` ×1 |
| Engine | POSIT only |
| Compounds with a donor N | 12, **all coordinating**, Fe–N 2.28–3.04 Å |
| …in the 2.0–2.4 Å crystal window | 3 |

Two substitutions were forced by the PoseBusters gate and are recorded in the selection
sheet. `OCNT-2313300` state 9 in 7UFB pressed into the porphyrin and was replaced by state 5
(2.34 Å). For `OCNT-2395712`, **eight of its nine poses failed** — every pose shorter than
3.04 Å reached the iron only by pushing the scaffold into the porphyrin — so the single clean
pose at 3.04 Å was used. That compound does not properly coordinate, but the reason is steric
rather than electronic: the scaffold cannot reach the axial site without a violation.

Each chosen pose was recombined with **its own** receptor's protein and heme, never with a
common reference frame, because a pose generated in one receptor and written into another is
scored against side chains it was never docked against.

## 7. What a correct pose looks like, measured

Nine deposited CYP3A4 complexes were used to calibrate the acceptance criteria, with the
geometry measured rather than assumed. The modes separate on two quantities:

| | coordinating (6 entries) | non-coordinating (3) |
|---|---|---|
| Fe–N | **1.94–2.42 Å** | 5.02–6.09 Å, or no donor |
| iron out of the donor ring plane | **0.15–0.88 Å** | 2.41–5.23 Å |
| donor ring tilt from edge-on | **1–14°** | 52–59° |
| closest ligand C above the porphyrin | **2.56–3.03 Å** | **3.46–3.97 Å** |
| ligand atoms within 4 Å of the porphyrin core | **14–27%** | 2–9% |

References include ketoconazole (`2V0M`, 2.10 Å), clotrimazole (`8SPD`, 1.94 Å) and bare
imidazole (`4D6Z`, 2.42 Å, 1.93 Å data — iron 0.36 Å off the ring plane, Fe–N–C 125/125°).

Two of these numbers did real work. **The last row discriminates on its own**: a coordinating
ligand is pulled down onto the heme by its ≈2 Å bond, so a pose showing a short Fe–N while its
carbons sit 3.5 Å above the porphyrin is internally inconsistent. And **contacting the
phenylalanine cluster proves nothing about the mode** — coordinating ligands touch 2–6 of
Phe-213/215/304, Ala-305, Thr-309, Ile-369 and non-coordinating ligands 5–6 — because a
type-II ligand still has to fill the cavity once its nitrogen is bound.

Accordingly, non-coordinating compounds were judged on **occupancy of the P1 and P2
hydrophobic subpockets** flanking the Phe cluster and the heme, at 3.3–3.8 Å, and *not* on
stacking flat against the porphyrin, which no deposited structure in this panel does.

## 8. Validation of the submitted file

| Check | Result |
|---|---|
| Organisers' structure validator | **valid**, no errors |
| PoseBusters (22 checks per complex) | **20 / 20 fully clean** |
| Structures present | 20, none missing or unexpected |
| Protein sequence | **0 mismatches** against UniProt P08684 over the resolved range |

The complexes carry 462 resolved residues (19 files, `7UFB`) or 463 (1 file, `6MA7`),
numbered 26/27–497 — the standard N-terminally truncated CYP3A4 construct — and every
resolved residue matches UniProt P08684.

Fe–N contacts at 2.28–3.04 Å pass `minimum_distance_to_inorganic_cofactors` — PoseBusters
does not treat genuine coordination as a clash.

One packaging detail is worth recording for other entrants: the validator reads the ligand
back and matches it to the expected SMILES with RDKit's
`AssignBondOrdersFromTemplate`, and with explicit ligand hydrogens present that match failed on
8 of our 20 ligands even though the heavy-atom graph was provably correct in every case
(verified by formula, heavy–heavy bond count and per-element degree signature). Stripping the
ligand hydrogens resolves it. We could not establish the mechanism; the effect is
reproducible.

## 9. Known limitations

- The receptor panel is effectively three distinct structures, not ten (§3), which caps what
  multi-receptor docking can contribute.
- POSIT's template machinery barely engaged (§5), so most poses come from template-free
  search — the component of POSIT with the least advantage over any other docking program.
  The one exception is instructive: in `6MA6`/`6MA7`, whose crystal ligands are themselves
  coordinating azoles, every returned pose came from the template-biased HYBRID engine.
- No deposited CYP3A4 complex contains a nitrile ligand, so nitrile coordination has no
  template and no yardstick. A constructed ideal nitrile pose is sterically feasible in the
  roomier receptors, but no docked pose approaches it; in any case all three nitrile-bearing
  compounds also carry a pyridine nitrogen, which is the better donor, so the question is moot
  for this set.
- One compound, `OCNT-2395712`, is submitted non-coordinating at 3.04 Å because its eight
  shorter poses all fail PoseBusters (§6.1).
- Selection remained the bottleneck throughout, and the final selection was made by eye —
  which §10.1 shows was the wrong choice.


---

## 10. Final result, and the main finding

**LDDT-PLI 0.3608, rank 34 / 38, Tier 2 of 4.** Coverage 1.0, so no structure was zeroed.

### 10.1 Expert visual selection lost to a one-line geometric rule

Four submissions were made from overlapping pose pools, differing only in how the pose was
chosen:

| submission | selection rule | LDDT-PLI |
|---|---|---|
| `24_4_posit_feN` | **automated: shortest free aromatic N to Fe** | **0.3939** |
| `24_11_frozen` | manual visual, 951 poses, 7UFB | 0.3381 live / 0.3608 final |
| `24_9_manual` | manual visual, 2,346 poses, 3 receptors | 0.3287 |
| `24_5_prob` | POSIT probability alone | 0.2894 |

Two independent rounds of careful visual inspection — by a medicinal chemist, with measured
Fe–N distances, crystal references and PoseBusters verdicts to hand — both scored **below** a
rule that fits on one line. This was not the result we expected and it is the most useful
thing the track produced.

The top and bottom rows form a controlled A/B over the same pose pool with one variable
changed, isolating coordination geometry at **+0.1045 LDDT-PLI**. We are not aware of many
prospective, blind tests of a pose-selection heuristic at this granularity.

A caveat on the comparison: `24_4` was scored only on the live board and `24_11` on both, and
the two scorings differ (`24_11` moved 0.3381 → 0.3608 for an identical file). The ranking of
the four is robust to that shift, but the exact gap is not.

### 10.2 Per-compound results

The challenge's head-to-head endpoint exposes per-compound LDDT-PLI, so the full
20 × 38 matrix is reconstructible. We placed in the field's top two on three compounds:

| compound | ours | field median | field best | rank | donor N |
|---|---|---|---|---|---|
| `OCNT-2312966` | **0.6183** | 0.2128 | **0.6183** | **1 / 38** | no |
| `OCNT-2313397` | 0.5048 | 0.4138 | 0.5069 | 2 / 38 | yes, 2.48 Å |
| `OCNT-2535519` | 0.5082 | 0.3464 | 0.5288 | 2 / 38 | yes, 2.57 Å |

`OCNT-2312966` — the 2-oxindole with no coordinating nitrogen, and the compound the field
found hardest of all twenty — received the best pose anyone submitted. Three top-two finishes
against an expectation of 1.05 gives p = 0.085, which is suggestive and not significant.

We were in the bottom decile on five compounds, so there is no class of pose on which we
outperformed as a group: both we and the field score higher on type-II-capable compounds than
on the rest, and our margin behind the winner is *larger* on the type-II compounds
(−0.151) than on the others (−0.077).

### 10.3 A pattern we checked and rejected

Our per-compound percentile correlates at **−0.527 (p = 0.017)** with how well the field did
on each compound — the most negative value among all 38 entrants, with only 2 of 38 reaching
significance. The natural reading is that a rigid geometric rule is relatively stronger where
template-driven methods struggle.

**We do not believe it.** Across entrants, mean score and that correlation are themselves
related at +0.520 (p = 0.0008): any below-average entrant gets a negative value
automatically, because scoring flatly ranks better where everyone scores low. Our standard
deviation across compounds is 4th lowest of 38, but mean and SD correlate at +0.615 across
entrants and our coefficient of variation is unremarkable. Selecting the most extreme of 38
entrants also makes p = 0.017 roughly what chance would produce. Reported because the
analysis was done, and the negative conclusion is the useful part.

### 10.4 Ground truth

The blinded Cryo-EM structures were published to the HuggingFace dataset
`openadmet/cyp-challenge-train-test` (`structures_unblinded/`) on 2026-11-05, without an
announcement — the changelog still states the structure ground truth is withheld. They are
phenix real-space-refined models at 3.50 Å, and the `LIG` heavy-atom count matches the
challenge SMILES for all 20 compounds. Section 11 scores every submission against them.


---

## 11. Ligand RMSD against the released ground truth

### 11.1 Method

The six submissions used **different crystal receptors**, so the ligands are not in a common
frame and a naive RMSD would be meaningless. For each compound and submission the submitted
protein is **superposed onto the ground truth** by Kabsch on shared Cα atoms (≥100 required),
the ligand is carried through that transform, and the comparison uses
`rdMolAlign.CalcRMS` — symmetry-corrected, and deliberately **not** superposing, since
`GetBestRMS` would align the ligand onto its target and report conformer similarity while
discarding the placement error.

Several ground-truth entries hold 2–3 protein chains and up to **6 ligand copies**
(`OCNT-2313300` has two sites per chain). Every copy is scored after superposing on its own
chain and the best is reported, as the challenge's symmetry-corrected BiSyRMSD does.

Both ligands are built by transferring the challenge SMILES topology onto the PDB
coordinates rather than by `AssignBondOrdersFromTemplate`: RDKit's proximity bonding on a
3.5 Å model perceives extra bonds — 31 bonds and a 5-valent carbon for `OCNT-2312966` — and
since the template is then a *subgraph*, the match succeeds while the spurious bonds survive
and sanitisation fails. Matching single-bonded graphs and copying coordinates works on all 20.

### 11.2 Validation

For the four submissions that were scored, our mean RMSD orders them **exactly** as the
organisers did — Spearman **+1.000** against BiSyRMSD and **−1.000** against LDDT-PLI, mean
absolute difference from official BiSyRMSD **0.20 Å**. The residual gap is expected, since
BiSyRMSD superposes on the binding site rather than globally.

### 11.3 The table

Ligand RMSD to ground truth, Å. Best per compound in bold.

| compound | donor N | 24_4 Fe-N rule | 24_5 probability | 24_8 deep Fe-N | 24_9 manual | 24_10 hybrid | 24_11 frozen |
|---|---|---|---|---|---|---|---|
| `OCNT-2312554` | yes | **4.41** | 7.25 | 5.96 | 7.10 | 5.96 | 4.82 |
| `OCNT-2312966` | no | 6.46 | 6.46 | **2.17** | 5.30 | **2.17** | **2.17** |
| `OCNT-2313038` | yes | **1.14** | 8.61 | **1.14** | **1.14** | **1.14** | 4.95 |
| `OCNT-2313300` | yes | **2.42** | 3.79 | 3.89 | 4.33 | 3.89 | 4.56 |
| `OCNT-2313397` | yes | 3.98 | 3.98 | 3.88 | **3.28** | 3.88 | **3.28** |
| `OCNT-2313429` | no | **5.79** | 9.27 | 6.16 | 6.47 | 6.16 | 6.58 |
| `OCNT-2315379` | no | 5.30 | 5.30 | **3.93** | **3.93** | **3.93** | **3.93** |
| `OCNT-2395328` | no | **5.07** | 5.13 | 5.15 | 5.66 | 5.15 | 5.66 |
| `OCNT-2395342` | no | 5.80 | 5.80 | **5.66** | **5.66** | **5.66** | **5.66** |
| `OCNT-2395415` | yes | 5.46 | 4.53 | **3.73** | 5.21 | **3.73** | 3.94 |
| `OCNT-2395436` | yes | 4.09 | 3.36 | 3.61 | 4.46 | **2.92** | 4.46 |
| `OCNT-2395457` | yes | 5.03 | **2.82** | 5.29 | 7.65 | 5.29 | 3.90 |
| `OCNT-2395460` | no | **4.42** | **4.42** | 6.60 | 4.88 | 6.60 | 6.57 |
| `OCNT-2395464` | yes | 4.17 | 4.17 | 3.41 | 6.34 | 6.34 | **3.06** |
| `OCNT-2395503` | yes | **1.69** | 3.95 | 3.52 | 3.75 | 3.52 | 4.00 |
| `OCNT-2395645` | no | 2.80 | 2.80 | **2.57** | **2.57** | **2.57** | **2.57** |
| `OCNT-2395712` | yes | 4.72 | 7.13 | 4.72 | **2.77** | 4.72 | 4.72 |
| `OCNT-2395789` | yes | 4.42 | 4.42 | 4.35 | **3.91** | 3.94 | 3.91 |
| `OCNT-2535422` | no | 4.37 | 4.37 | **4.00** | 5.99 | **4.00** | 5.99 |
| `OCNT-2535519` | yes | 4.25 | 7.03 | **2.08** | 6.58 | **2.08** | 3.48 |
| **mean** | | **4.29** | **5.23** | **4.09** | **4.85** | **4.18** | **4.41** |
| **median** | | 4.41 | 4.47 | 3.91 | 5.04 | 3.94 | 4.23 |
| **within 2 Å** | | 2 | 0 | 1 | 1 | 1 | 0 |
| **official LDDT-PLI** | | 0.3939 | 0.2894 | not submitted | 0.3287 | not submitted | 0.3608 (final) |


### 11.4 The Fe–N effect now has a confidence interval

§6 measured the coordination rule at +0.1045 LDDT-PLI from a single blind A/B with no
interval, because per-compound scores did not exist. They do now:

| | |
|---|---|
| `24_4` Fe–N rule | mean RMSD **4.29 Å** |
| `24_5` probability only | mean RMSD **5.23 Å** |
| difference | **+0.94 Å**, 95% CI **[+0.11, +1.91]** (20,000 bootstrap resamples) |
| paired Wilcoxon | **p = 0.041** |

Nine of twenty poses are identical between the two; of the **11 that differ** the Fe–N rule
is better on **8**, mean difference **+1.71 Å** (p = 0.042). The largest single gain is
`OCNT-2313038`, **1.14 Å against 8.61 Å**. The interval is wide — twenty compounds — but it
excludes zero.

### 11.5 The two unsubmitted candidates were better, but not significantly

| submission | mean RMSD | vs `24_4` | better on | paired p |
|---|---|---|---|---|
| **`24_8` deep Fe–N** *(never submitted)* | **4.09** | **−0.20** | 11/20 | 0.557 |
| **`24_10` hybrid** *(never submitted)* | **4.18** | −0.11 | 10/20 | 0.845 |
| `24_4` Fe–N rule | 4.29 | — | — | — |
| `24_11` frozen *(final)* | 4.41 | +0.12 | 10/20 | 0.778 |
| `24_9` manual | 4.85 | +0.56 | 8/20 | 0.117 |
| `24_5` probability | 5.23 | +0.94 | 3/20 | **0.041** |

Nominally better, statistically indistinguishable — consistent with this leaderboard being
unable to resolve differences below ≈0.1 LDDT-PLI. The one significant row is the bottom
one.

### 11.6 The honest summary

Only **2 of 120** compound–submission pairs land within 2 Å, and the best mean over the whole
set is 4.09 Å. Even `OCNT-2312966`, where we placed first of 38 on LDDT-PLI, is 2.17 Å in our
best submissions. None of our submissions placed most ligands correctly; the differences
between them are differences between degrees of wrongness.
