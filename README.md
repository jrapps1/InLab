# InLab — LIS Training Tools for Microbiology Accessioning

Interactive browser-based training tools for Medical Laboratory Technology (MLT) students at the College of New Caledonia. Students practice specimen accessioning decisions — reviewing requisition forms, specimen labels, container and transport conditions, and selecting appropriate culture media — in a realistic LIS interface.

No installation required. Open either HTML file directly in a browser.

---

## Files

| File | Purpose |
|------|---------|
| `lis-practice.html` | Self-directed practice with immediate per-case feedback |
| `lis-simulation.html` | Instructor-configured assessment with CSV export for grading |

---

## lis-practice.html — Student Practice Mode

### What it does

Generates a randomized set of microbiology accessioning cases. For each case the student sees:

- A handwritten-style **requisition form** (patient name, DOB, PHN, ordering physician, source, tests requested, collection date)
- A printed **specimen label** (patient name, PHN/ID, collection date and time)
- **Specimen details** panel showing the observed container, transport condition, and quantity/appearance

The student decides to **Accept** or **Reject** the specimen. If rejecting, they select one or more reasons from a categorized checklist (Label Issues, Requisition Issues, Specimen Suitability). Accepted specimens proceed to a **media selection** step where the student chooses the correct culture plates and stains.

After each case the student receives instant feedback showing:
- Whether the accept/reject decision was correct
- Which rejection reasons were correctly identified, missed, or spurious
- Which culture media were correctly selected (for accepted specimens)

A session summary at the end shows all results in a table.

### How to use

1. Open `lis-practice.html` in a browser.
2. Enter any Tech ID (e.g. your student number) and choose a case count (5–30).
3. Work through each case and review your feedback after each one.
4. Review the session summary when complete.

### Case distribution

Cases are randomly generated with an approximate distribution:

| Category | ~% of cases |
|----------|-------------|
| Acceptable (no errors) | 15% |
| Label errors only | 25% |
| Requisition errors only | 25% |
| Specimen suitability errors | 20% |
| Mixed errors | 15% |

### Specimen types covered

Urine, Blood Culture, Wound Swab, Sputum, Stool, Throat Swab, Nasopharyngeal Swab, CSF, Tissue

---

## lis-simulation.html — Instructor Simulation Mode

### What it does

Instructors build a fixed set of cases in advance; students work through them without seeing any per-case feedback. Results are exported as a CSV for marking.

### Instructor setup

1. Open `lis-simulation.html` in a browser.
2. Click the **Instructor** button in the header.
3. Enter the instructor password (set near the top of the `<script>` block — change it before distributing the file).
4. Click **Add Case** and configure each case:
   - Choose specimen type, patient name (optional — random if blank), container, transport condition, and appearance notes
   - Check all errors that are present and should be flagged as rejection reasons
5. Click **Save Case**. Repeat for as many cases as needed.
6. Click **Done** and distribute the file to students.

> Cases are stored in the browser's `localStorage` under the key `sjmh_sim_cases`. Students must use the **same browser and computer** where the cases were configured, or the instructor must export/share the file with cases already saved in it (cases are embedded in localStorage, not the file — see note below).

> **Note:** To share pre-built cases between computers, open the browser console after building cases and copy the value of `localStorage.getItem('sjmh_sim_cases')`. Students can paste it back via `localStorage.setItem('sjmh_sim_cases', '<paste>')` before starting their session. A future update may embed cases directly in the HTML file.

### Student workflow

1. Open the configured `lis-simulation.html`.
2. Enter a Tech ID.
3. Work through all cases — no feedback is shown during the session.
4. On the summary screen, click **Export CSV** and submit the file to the instructor.

### CSV export columns

Tech ID, Case, Specimen Type, Correct Verdict, Student Decision, Accession Result, Missed Reasons, Extra Reasons, Plating Result, Correct Plates, Student Plates, Missed Plates, Extra Plates, Session Date

---

## Rejection reasons and scoring

Both tools use the same rejection reason checklist, organized into three categories:

**Label Issues** — missing name, missing identifier (PHN), missing date, missing time, name mismatch, PHN mismatch

**Requisition Issues** — missing patient name, DOB, PHN, tests requested, specimen source, collection date, ordering provider

**Specimen Suitability** — incorrect container, insufficient quantity, transported incorrectly, stability compromised (age), stability compromised (temperature), inadequate fixative, specimen too large for container, leaking specimen

### Scoring equivalences

Some pairs of reasons are clinically interchangeable and both are accepted as correct:

- **"Transported incorrectly"** and **"Stability compromised (temperature)"** — a wrong transport temperature is simultaneously a transport error and a temperature-related stability issue
- **"Incorrect container"** and **"Inadequate fixative"** — formalin used on a microbiology tissue specimen is both a wrong container and an inadequate fixative for culture

---

## Media selection (plating)

When a specimen is accepted, students select the culture media and stains for that specimen type. Selections are scored against the lab procedure manual for Saint Jason Memorial Hospital.

| Specimen | Required media |
|----------|---------------|
| Urine (midstream/catheter) | BAP O₂, MAC |
| Urine (suprapubic aspirate) | GM, BAP CO₂, CAP CO₂, MAC |
| Blood Culture | Loaded directly to BACTEC incubator — no plating |
| Wound Swab | GM, BAP CO₂, MAC |
| Sputum | GM, BAP CO₂, CAP CO₂, MAC |
| Stool | BAP O₂, MAC, SMAC, XLD, HE, CIN 30°, CAMP 42° |
| Throat Swab | BAP ANA |
| Nasopharyngeal Swab | Forwarded to Virology/Molecular — no plating |
| CSF | GM, BAP CO₂, CAP CO₂, MAC, THIO (STAT) |
| Tissue | GM, BAP CO₂, CAP CO₂, MAC, BAP ANA, THIO (± BBE ANA, PEA ANA conditional) |

---

## Customization

All styling uses CSS custom properties defined in `:root`. The primary brand colour is `--cnc-red: #C8102E`. Font families are Inter (sans-serif), JetBrains Mono (monospace), and Caveat (handwritten form values).

The instructor password in `lis-simulation.html` is defined as `const INSTR_PASSWORD` near the top of the `<script>` block.
