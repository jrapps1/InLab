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

**Practice:** `screen-login` → `screen-case` → `screen-feedback` (per case) → `screen-summary`

**Simulation:** `screen-login` → `screen-case` → `screen-summary` (no per-case feedback)

### Shared data layer

Both files duplicate these data structures (changes must be made in both):

- `SPECIMENS` — array of 9 specimen type objects (`Urine`, `Blood Culture`, `Wound Swab`, `Sputum`, `Stool`, `Throat Swab`, `Nasopharyngeal Swab`, `CSF`, `Tissue`). Each has `validContainers`, `wrongContainers`, `validTransport`, `wrongTransport`, `maxHours`, etc.
- `ERROR_META` — flat dict mapping error keys (e.g. `label_no_name`, `specimen_wrong_container`) to `{ cat, desc }`. Categories are `label`, `req`, `specimen`.
- `MEDIA_GROUPS` / `getPlatingSpec(specType, source)` — plating media definitions and per-specimen-type required/conditional plate lookup.

### Case object shape

After generation/building, each case carries: `label` (what's on the specimen label), `req` (requisition fields), `container`, `transport`, `qty`, `specNotes`, `errors` (array of error keys), `verdict` (`'ACCEPT'`|`'REJECT'`), `correctReasons`.

Missing field values are rendered as empty strings; the UI renders them as blank handwriting-font lines using the `.req-field-value.missing` / `.label-val.missing` CSS classes.

### Case generation

**Practice (`generateOneCase`)** — randomises all patient/specimen details and injects errors according to a category (`ok`, `label`, `req`, `specimen`, `mixed`). Distribution in `generateCases`: ~15% ok, ~25% label, ~25% req, ~20% specimen, ~15% mixed.

**Simulation (`buildRenderableCase`)** — reads a pre-built config object from `localStorage` (key `sjmh_sim_cases`) and applies the instructor-specified errors to otherwise-random patient data.

### Decision and plating flow

1. Student selects Accept or Reject; Reject reveals reason checkboxes.
2. On submit, `submitCase()` computes `score` (`correct` / `partial` / `incorrect`) and stores `state.pendingDecision`.
3. If Accept, `showPlatingStep()` renders media checkboxes. `submitPlating()` evaluates selected vs. required plates.
4. `finaliseCase()` writes to `state.results` and advances to the next case (or summary).

## Design conventions

- CSS custom properties in `:root` control the entire colour scheme. `--cnc-red: #C8102E` is the brand colour.
- Three font families: `--sans` (Inter), `--mono` (JetBrains Mono), `--hand` (Caveat) — Caveat gives the handwritten look to requisition and label values.
- `rot()` / `slightRotation()` helpers apply a tiny random CSS rotation to handwritten text to simulate real writing.
- No framework, no modules — all JS is vanilla, `'use strict'`, in a single `<script>` block.
- `state` is a plain object holding session data; there is no reactive system.

## Simulation instructor access

The instructor password is defined as a constant near the top of the `<script>` block in `lis-simulation.html`. Change it there before distributing the file to students.
