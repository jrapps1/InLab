# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Two standalone single-file HTML tools for MLT (Medical Laboratory Technology) students at College of New Caledonia. They simulate a hospital LIS (Laboratory Information System) for microbiology specimen accessioning practice. No build process, no package manager — open directly in a browser.

- **lis-practice.html** — self-directed practice; cases are randomly generated, and per-case feedback is shown immediately after each submission
- **lis-simulation.html** — instructor-controlled assessment; the instructor pre-builds cases via a password-protected modal, cases persist in `localStorage`, no per-case feedback is shown to the student, results are exported as CSV

## Architecture

Each file is a self-contained SPA with three sections in order: `<style>`, HTML screens, `<script>`.

### Screen flow

Both files use a `.screen` / `.screen.active` pattern. `showScreen(id)` deactivates all screens and activates the target.

**Practice:** `screen-login` → `screen-worklist` → `screen-case` → `screen-feedback` (per case) → `screen-summary`

**Simulation:** `screen-login` → `screen-case` → `screen-summary` (no per-case feedback)

### Shared data layer

Both files duplicate these data structures (changes must be made in both):

- `SPECIMENS` — array of 9 specimen type objects (`Urine`, `Blood Culture`, `Wound Swab`, `Sputum`, `Stool`, `Throat Swab`, `Nasopharyngeal Swab`, `CSF`, `Tissue`). Each has `validContainers`, `wrongContainers`, `validTransport`, `wrongTransport`, `maxHours`, etc.
- `ERROR_META` — flat dict mapping error keys (e.g. `label_no_name`, `specimen_wrong_container`) to `{ cat, desc }`. Categories are `label`, `req`, `specimen`.
- `EQUIVALENT_GROUPS` — array of error-key arrays treated as interchangeable for both scoring and feedback display. Currently: `['specimen_transport_error', 'specimen_stability_temp']` and `['specimen_wrong_container', 'specimen_inadequate_fixative']`. Scoring and feedback both use `getEquivalents(key)` — do not add raw `includes` checks against error keys where equivalents exist.
- `MEDIA_GROUPS` / `getPlatingSpec(specType, source)` — plating media definitions and per-specimen-type required/conditional plate lookup.

### Case object shape

After generation/building, each case carries: `label` (what's on the specimen label), `req` (requisition fields), `container`, `transport`, `qty`, `specNotes`, `errors` (array of error keys), `verdict` (`'ACCEPT'`|`'REJECT'`), `correctReasons`, `specStandardQty`, `specRequiredContainer`, `stabilityHours`, `stabilityMaxHours`.

Missing field values are rendered as empty strings; the UI renders them as blank handwriting-font lines using the `.req-field-value.missing` / `.label-val.missing` CSS classes.

### Case generation

**Practice (`generateOneCase`)** — randomises all patient/specimen details and injects errors according to a category (`ok`, `label`, `req`, `specimen`, `mixed`). Distribution in `generateCases`: ~30% ok, ~20% label, ~20% req, ~15% specimen, ~15% mixed.

**Key invariants to maintain when editing case generation:**

- **Effective collection date** — `specimen_stability_age` shifts the collection timestamp. `effectiveCollected` must be computed from this *before* label/req fields are built, so `label_no_date`/`label_no_time` are never silently overwritten by a later mutation.
- **Mutually exclusive label errors** — `label_no_name` and `label_name_mismatch` must not co-occur (a mismatch is unverifiable when the name field is blank); same for `label_no_identifier` / `label_phn_mismatch`. The label pool is partitioned into sub-groups (name, identifier, date/time) and at most one is picked from each.
- **`specimen_stability_temp` is not injected** — it is kept as a student-selectable equivalent of `specimen_transport_error` via `EQUIVALENT_GROUPS`, but is not placed in any error generation pool. Injecting both in the same case would produce contradictory feedback.
- **`specimen_too_large` is specimen-type restricted** — only applicable to solid/large-volume specimens: Stool, Sputum, and Tissue.
- **Blood Culture Aerobic-only orders** — when the ordered test is `'Aerobic Blood Culture'`, `container`, `specRequiredContainer`, and `specStandardQty` are all updated to reflect a single aerobic bottle (not the aerobic+anaerobic set).

**Simulation (`buildRenderableCase`)** — reads a pre-built config object from `localStorage` (key `sjmh_sim_cases`) and applies the instructor-specified errors to otherwise-random patient data.

### Decision and plating flow

1. Student selects Accept or Reject; Reject reveals reason checkboxes.
2. On submit, `submitCase()` computes `score` (`correct` / `partial` / `incorrect`) and stores `state.pendingDecision`.
3. If Accept, `showPlatingStep()` renders media checkboxes. `submitPlating()` evaluates selected vs. required plates.
4. `finaliseCase()` writes to `state.results` and advances to the next case (or summary).

Scoring uses `getEquivalents()` throughout — selecting either member of an equivalent pair is accepted. The feedback display also uses `getEquivalents()` (not a raw `includes`) so that the "✓ Identified / ✗ Missed" tags are consistent with the score.

### Plating notes

- **Suprapubic aspirate (Urine)** — requires `GM, BAP CO2, CAP CO2, MAC O2, BAP ANA`. The anaerobic BAP is required because suprapubic aspirate is a sterile-site specimen.
- **Blood Culture** — no conventional agar plating; bottles go directly to the BACTEC incubator.
- **Nasopharyngeal Swab** — forwarded to Virology/Molecular; no conventional agar plating.

## Design conventions

- CSS custom properties in `:root` control the entire colour scheme. `--cnc-red: #C8102E` is the brand colour.
- Three font families: `--sans` (Inter), `--mono` (JetBrains Mono), `--hand` (Caveat) — Caveat gives the handwritten look to requisition and label values.
- `rot()` / `slightRotation()` helpers apply a tiny random CSS rotation to handwritten text to simulate real writing.
- No framework, no modules — all JS is vanilla, `'use strict'`, in a single `<script>` block.
- `state` is a plain object holding session data; there is no reactive system.
- PHN format is `XXXX-XXX-XXX` (10 digits, two hyphens). `altPHN()` mutates only the last 3-digit segment to preserve this format.

## Simulation instructor access

The instructor password is defined as a constant near the top of the `<script>` block in `lis-simulation.html`. Change it there before distributing the file to students.
