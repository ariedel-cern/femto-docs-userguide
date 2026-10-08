---
title: Femto Analysis Framework — User Guide
tags: femto, o2physics, documentation, userguide
slideOptions:
  theme: white
  transition: slide
---

# Femto Analysis Framework
## User Guide

PWGCF/Femto — O2Physics

---

## Scope of this guide

This is one of three documentation repositories:

- **femto-docs-code** — code-level documentation (architecture, class reference, how to extend the framework)
- **femto-docs-userguide** — *this repo*: how to configure and run the framework
- **femto-docs-deriveddata** — the derived data format: tables, columns, versioning

----

This deck covers the **userguide** part: enough to configure a production run and understand what comes out the other end, without needing to read the C++.

---

## What the framework does, in one slide

- Reads Run 3 AO2D data: tracks, V0s, cascades, kinks, PCM photons, MC truth, collisions
- Applies **configurable** quality and PID selections
- Writes compact **derived tables** ("Femto tables")
- Downstream tasks (pairing, event mixing, QA) consume *only* the derived tables, never the original AO2D

---

# Building particles & collisions in the producer

---

## Two stages of selection

Every particle species — and the collision itself — goes through the same two-stage pattern:

1. **Prefilters** — loose, mandatory kinematic/acceptance cuts
2. **Bits** — configurable quality/PID cuts, recorded as bits in a bitmask

----

Same pattern everywhere: tracks, V0s, cascades, kinks, photons, collisions. Learn it once, it applies to every builder.

---

## Stage 1: Prefilters

- Applied first, before anything else is even considered
- Cheap, loose, usually **mandatory** — fail a prefilter and the candidate is dropped, full stop
- Typical examples: pT range, |eta| range, phi range, a loose invariant-mass window
- Purpose: cut combinatorics early — don't spend bits (or CPU) on candidates nobody downstream will ever want

----

Prefilter values are themselves recorded in a **filter histogram** in the output, so the exact cuts used are reproducible from the ROOT file alone, without needing the original run configuration.

---

## Stage 2: Bits and bitmasks

- For every quality/PID cut, you configure **one or more thresholds**, from loosest to tightest
- Each threshold occupies **one bit**
- All bits for all cuts of a species are packed into a **single integer** — the selection *mask*

---

## Why bitmasks? → Compression

This is the central design goal of the whole framework:

> **Produce once, select many times, downstream.**

- Storing one integer mask per candidate is far cheaper than storing N float columns × M analyses
- One production run yields one set of bits
- Every downstream analysis re-slices the *same* bits with a *different* required mask — no need to re-run the producer just to tighten a cut

---

## Example: a track's bitmask (schematic)

Exact bit layout lives in the code docs — this is just the idea:

```
bit 0   : TPC clusters >= loosest threshold
bit 1   : TPC clusters >= tighter threshold
bit 2   : |DCAxy| < loosest threshold
...
bit N   : TOF nSigma_proton < tightest threshold
```

A downstream task then just checks:

```
(mask & requiredBits) == requiredBits
```

---

## Mandatory vs. optional bits

Not every bit treats a failure the same way:

- **Minimal (mandatory)** cut — fails ⇒ candidate is rejected outright, dropped from the table entirely, not just left unselected
- **Optional** cut — candidate survives if it passes *any one* cut in its optional group (OR-logic — e.g. "PID via ITS *or* TPC *or* TOF")
- Everything else — just a recorded bit, no effect on whether the candidate survives at all

----

Getting mandatory vs. optional right matters: a cut that's meant to be a hard requirement but is wired as neither will silently let everything through.

---

## Gates that don't show up as bits

Some configuration values don't become a cut themselves, but change what an existing bit *means* — e.g. a momentum threshold above which TOF PID becomes mandatory, or a flag that decides whether a daughter track without a TOF hit is auto-passed or auto-rejected.

The framework records these as a **comment** attached to the relevant bit's histogram label, so the output stays self-documenting even for these "hidden" switches.

---

## TODO / next sections

- [ ] Collision builder & event selection bits
- [ ] Track builder (quality cuts, PID, TOF thresholds)
- [ ] V0 / Cascade / Kink builders
- [ ] Photon (PCM) builder
- [ ] Pair & triplet building, event mixing
- [ ] Configuring a run: JSON structure walkthrough
- [ ] Reading the output: selection & filter histograms

---

# Thanks!

Questions → PWGCF/Femto maintainers
