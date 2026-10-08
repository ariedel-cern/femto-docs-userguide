# Femto Analysis Framework — User Guide

## Scope of this document

This is one of three documentation repositories for the PWGCF/Femto analysis framework
(O2Physics):

- **femto-docs-code** — code-level documentation (architecture, class reference, how to
  extend the framework with a new particle species or selection)
- **femto-docs-userguide** — *this repo*: how to configure and run the framework
- **femto-docs-deriveddata** — the derived data format: tables, columns, versioning

This document covers the user-guide part: enough to configure a production run and
understand what comes out the other end, without needing to read the C++. It is written as
plain reference text rather than a slide deck — slides built from this material (if any)
live elsewhere.

## What the framework does

The framework reads Run 3 AO2D data — tracks, V0s, cascades, kinks, PCM photons, MC truth,
and collisions — applies configurable quality and PID selections to each, and writes out
compact derived tables ("Femto tables"). Downstream tasks such as pair and triplet building,
event mixing, and QA consume only these derived tables; they never touch the original AO2D
data directly.

<!-- TODO: diagram of the overall data flow, AO2D -> producer -> Femto tables -> downstream tasks -->

## Building particles and collisions in the producer

### Two stages of selection

Every particle species — and the collision itself — goes through the same two-stage
pattern in the producer:

1. **Prefilters** — loose, mandatory kinematic and acceptance cuts.
2. **Bits** — configurable quality and PID cuts, recorded as bits in a bitmask.

This pattern repeats across every builder (tracks, V0s, cascades, kinks, photons,
collisions), so it is worth understanding once in general terms before looking at any
individual species.

### Stage 1: Prefilters

Prefilters are applied first, before anything else is considered. They are cheap, loose,
and usually mandatory: if a candidate fails a prefilter, it is dropped outright rather than
merely left unselected. Typical examples are a pT range, an |eta| range, a phi range, or a
loose invariant-mass window.

The purpose of this stage is to cut combinatorics early, so that CPU time and, more
importantly, bitmask space are not spent on candidates that no downstream analysis would
ever want anyway.

The prefilter values themselves are recorded in a filter histogram in the output, so the
exact cuts used in a given production are reproducible directly from the ROOT file, without
needing the original run configuration.

### Stage 2: Bits and bitmasks

For every quality or PID cut, the user configures one or more thresholds, ordered from
loosest to tightest. Each threshold occupies one bit. All the bits for all the cuts of a
given species are packed into a single integer — the selection mask.

### Why bitmasks: compression

This is the central design goal of the whole framework: produce once, select many times,
downstream. Storing one integer mask per candidate is far cheaper than storing N separate
float columns for every cut, multiplied by every analysis that might want a slightly
different threshold. A single production run yields one fixed set of bits; every downstream
analysis then re-slices those same bits against a different required mask, without ever
needing to re-run the producer just to tighten or loosen a cut.

### Example: a track's bitmask

The following is schematic only — the exact bit layout for each species lives in the code
documentation, not here:

```
bit 0   : TPC clusters >= loosest threshold
bit 1   : TPC clusters >= tighter threshold
bit 2   : |DCAxy| < loosest threshold
...
bit N   : TOF nSigma_proton < tightest threshold
```

A downstream task then simply checks `(mask & requiredBits) == requiredBits`.

<!-- TODO: picture of an actual bitmask layout for a track, with bit offsets labeled -->

### Mandatory vs. optional bits

Not every bit treats a failure the same way:

- A **minimal (mandatory)** cut rejects the candidate outright on failure — it is dropped
  from the table entirely, not just left unselected.
- An **optional** cut accepts the candidate if it passes *any one* cut in its optional
  group. This implements OR-logic across a group of cuts, for example "PID via ITS, or TPC,
  or TOF" for a single species.
- Any other bit is simply recorded, with no effect on whether the candidate survives.

Getting mandatory vs. optional right matters: a cut that is meant to be a hard requirement
but is wired as neither mandatory nor optional will silently let every candidate through
regardless of whether it actually passes.

### Gates that don't show up as bits

Some configuration values do not become a cut in their own right, but change what an
existing bit *means*. A momentum threshold above which TOF PID becomes mandatory is one
example; a flag that decides whether a daughter track without a TOF hit is auto-passed or
auto-rejected is another. The framework records these as a comment attached to the relevant
bit's histogram label, so the output stays self-documenting even for these less obvious
switches.

## Still to be written

- Collision builder and event selection bits
- Track builder (quality cuts, PID, TOF thresholds)
- V0 / Cascade / Kink builders
- Photon (PCM) builder
- Pair and triplet building, event mixing
- Configuring a run: JSON structure walkthrough
- Reading the output: selection and filter histograms
