# RH minimal-import audit (work in progress)

This audit belongs only to branch `audit/minimal-rh-verification-20261121`.
It does **not** alter the original `Main.lean`, any original proof module, or the original
`Challenge.lean`. These two audit entrypoints are **not verified as compiled**.

## What was established through source inspection

* `Main.lean` imports `Supplements.Kernels.MainTheoremsAll`.
* `MainTheoremsAll.lean` imports `MainTheoremsCritical.lean` and `CorGKOf.lean`.
* `MainTheoremsCritical.lean` imports `MainTheorems.lean` and three other
  small-class-number modules.
* `MainTheorems.lean` defines `rhOfAll`, `allChi`, `critAll`, and `claimAll`,
  which are precisely the four declarations referenced in `Main.lean`.
* `CharacterBoxes/MainConclusion.lean` defines
  `UpperTheory.FinalTheorems.riemannHypothesis : RiemannHypothesis`.
* Therefore `AuditRH.lean` and `AuditAll.lean` are candidates to remove **direct**
  imports that do not appear necessary for the stated results. Their complete import
  closures may still overlap heavily. No speed-up or reduction is guaranteed.

## Stage 1: compile on an isolated high-memory runner

Use pinned Lean `v4.33.1`; fetch dependencies in a controlled network phase,
then disable the network for compilation.

```bash
lake exe cache get
lake build Challenge AuditRH
lake build AuditAll
```

The audit sources contain `#print axioms` commands. In the successful build logs,
record the output for every stated theorem and make sure no unexpected axioms occur.
This is an auxiliary check only: an independent kernel replay is still necessary.

**Do not start this on ordinary GitHub-hosted runners.** The original verifier
estimates hundreds of GB of RAM for the most demanding independent checker.
Even the supposedly minimal modules still import the main mathematical proof.

## Stage 2: export and independent rechecking

After building `lean4export` tag `v4.33.0` with Lean `v4.33.1`, use the exact
`PRELUDE` and `Quot` arguments in the root README's "Running Comparator" section.
For the one-theorem test:

```bash
THEOREMS="riemannHypothesis"
# PRELUDE: copy from README
lake env lean4export Challenge -- Quot Quot.mk Quot.lift Quot.ind $THEOREMS $PRELUDE > challenge-rh.export
lake env lean4export AuditRH -- Quot Quot.mk Quot.lift Quot.ind $THEOREMS $PRELUDE > auditrh.export
```

For the five-theorem test:

```bash
THEOREMS="riemannHypothesis riemannZeta_eq_zero_re LFunction_ne_zero_of_half_lt_re LFunction_eq_zero_re LFunction_eq_zero_re_of_isPrimitive"
lake env lean4export Challenge -- Quot Quot.mk Quot.lift Quot.ind $THEOREMS $PRELUDE > challenge-all.export
lake env lean4export AuditAll -- Quot Quot.mk Quot.lift Quot.ind $THEOREMS $PRELUDE > auditall.export
```

With official Lean v4.36.0 and Comparator on the PATH, in a clean work directory:

```bash
lake comparator --config comparator-auditrh.json --challenge-from-export challenge-rh.export --solution-from-export auditrh.export
lake comparator --config comparator-auditall.json --challenge-from-export challenge-all.export --solution-from-export auditall.export
```

Retain export checksums, tool versions, full logs and peak RAM. Cross-check with the
unaltered `Main` export, not only the audit entrypoints.

## Current status (2026-11-21)

* Pinned Mathlib RH definition checked separately at the exact pinned revision.
* Upstream CI passed source provenance and compiling `Challenge` only.
* Original full proof build: no completed public result in repository at time of audit.
* These two audit entrypoints: **source-level proposal only; compilation not run**.
* Independent Comparator on the five theorem claims: **not run**.

Primary blocker: this environment has roughly 5.8 GiB RAM and 30 GiB free disk,
no installed Lean, and direct GitHub DNS access is unavailable.
No proof certificate has been accepted by an independent kernel here.
