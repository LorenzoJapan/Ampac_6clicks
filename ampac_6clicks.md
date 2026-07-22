# AM-PAC "6-Clicks" Calculator — Design Notes

Single-file HTML calculator for the Activity Measure for Post-Acute Care "6-Clicks" inpatient short forms. Bedside-ready, offline-capable, and self-contained — no build step, no dependencies, no PHI storage.

## What it does

Lets a clinician tap a 1–4 score for each item of the Basic Mobility or Daily Activity / Self-Care short form and computes raw totals automatically. The user chooses which short form(s) to score at the top; only the chosen section(s) render. Raw scores are interpreted against a simplified discharge-and-therapy guide that adapts to the selected form(s).

## Features

- **Mode selector.** Basic Mobility only, Daily Activity only, or Both.
- **Tap-to-score UI.** 4 large color-coded buttons (Unable / A Lot / A Little / No Assist) per item; tap a selected score again to clear.
- **Live totals.** Six-dot progress tracker and running raw score chip in each section header; chip turns teal when all 6 items are scored.
- **Score summary card.** Raw score /24, progress bar, and per-band interpretation (three bands per form).
- **Adaptive interpretation guide** in the summary card:
  - Basic Mobility selected → a **Basic Mobility** row only.
  - Daily Activity selected → a **Daily Activity/Self-Care** row only.
  - Both selected → both rows, a **Combined** rule row, and a live **This patient** readout that fills in once all 12 items are scored.
- **Copy scores.** Generates a plain-text score block for pasting into an EHR note, including the guide line(s) for the active form(s) and the combined rule in Both mode.
- **Vancouver-format references** with clickable DOIs, retained as background reading.
- **Auto-save.** Scores persist across page reloads via `localStorage`; Reset clears both the scores and the mode selection.
- **Accessible.** Radiogroup semantics on score rows, visible keyboard focus, `prefers-reduced-motion` respected.
- **Responsive.** Down to phone width.
- **No PHI.** No patient name, MRN, date, or evaluator fields.

## Scoring guide

| Score | Rating       | Clinical meaning                                        |
| :---: | :----------- | :------------------------------------------------------ |
|   1   | Unable       | Total / dependent assist (>75% assistance needed)       |
|   2   | A Lot        | Max / mod assist (50–75% assistance needed)             |
|   3   | A Little     | Min / contact guard assist / supervision (<25% assist)  |
|   4   | No Assistance| Modified independent / independent (no physical help)   |

Score based on observed performance or clinical judgment. Do not alter the intent of the item.

## Short-form items

### Basic Mobility

1. Turning over in bed
2. Lying down to sitting on edge of bed
3. Bed to chair (seated transfer)
4. Standing up from a chair
5. Walking in room
6. Climbing 3–5 steps with a railing

### Daily Activity / Self-Care

1. Feeding / eating meals
2. Personal grooming (teeth, face)
3. Upper body dressing (on & off)
4. Lower body dressing (on & off)
5. Bathing (washing, rinsing, drying)
6. Toileting (toilet, bedpan, or urinal)

## Interpretation scheme

Simplified, form-symmetric cutoffs used throughout the app:

| Form | 18–24 | ≤17 |
| :--- | :---- | :-- |
| Basic Mobility | Likely home | Consult PT; facility possible |
| Daily Activity/Self-Care | Likely home | Consult OT; facility possible |

**Combined rule** (Both mode): both ≥18 → home likely · either ≤17 → consult therapy · both ≤17 → facility likely.

### Summary-card bands (implemented in `interpText`)

| Score | Basic Mobility | Daily Activity |
| :---: | :------------- | :------------- |
| 22–24 | Independent mobility; PT usually not needed | Independent self-care; OT usually not needed |
| 18–21 | Mobile; likely home | Manages self-care; likely home |
| 6–17  | Needs mobility help; consult PT; facility possible | Needs self-care help; consult OT; facility possible |

### Live combined readout (`updateCombined`)

Rendered only in Both mode, only when both forms are complete:

| Condition | Readout |
| :--- | :--- |
| BM ≥18 and DA ≥18 | Home likely (adds "therapy usually not needed" when both ≥22) |
| Exactly one form ≤17 | Consult PT (if BM low) or OT (if DA low); facility possible |
| BM ≤17 and DA ≤17 | Consult PT & OT; facility likely |

### Relationship to the published literature

These cutoffs are a pragmatic simplification for bedside triage. The cited literature reports Basic Mobility–specific thresholds — ≤18 as the primary PT/OT-necessity cutoff,<sup>5,6,7</sup> ≤15 as a stronger OT / post-acute-care indicator,<sup>4</sup> and ≥23 as highly independent<sup>8,4</sup> — and no published Daily Activity cutoffs. The references are retained in the app for the underlying evidence, but the panel no longer displays per-claim citation chips.

## Architecture

Single-file HTML (`index.html`) with inline `<style>` and `<script>`. No external assets, no CDN calls, no network dependencies.

| File               | Purpose                                              |
| :----------------- | :--------------------------------------------------- |
| `index.html`       | The app itself (named for GitHub Pages root serving).|
| `ampac_6clicks.md` | This document (design notes + citations).            |

### State model

```js
state = {
  mode: "bm" | "da" | "both" | null,
  scores: {
    bm: [null|1|2|3|4, × 6],
    da: [null|1|2|3|4, × 6]
  }
}
```

Persisted to `localStorage` under the key `ampac6clicks_calc_v4`. The schema is unchanged from v4 (mode + scores), so the key was not bumped when the interpretation scheme was simplified. Reset clears both the persisted state and the DOM.

### Scoring math

Raw score = sum of six items (min 6, max 24). Section shown as `— / 24` until all six items are scored; if fewer than six are complete, a partial sum is displayed with an asterisk and the count of items scored. Progress-bar fill = `((raw − 6) / 18) × 100%`.

### Key functions

| Function | Role |
| :--- | :--- |
| `setMode` / `activeIds` | Mode selection; which sections render |
| `buildSections` | Renders the tap-to-score item cards |
| `buildSummary` | Renders result boxes; shows the interpretation panel for any active mode |
| `buildThresholds(ids)` | Renders the adaptive guide rows (per-form, plus Combined + This patient in Both mode) |
| `refresh` | Updates buttons, dots, totals, bands; calls `updateCombined` |
| `updateCombined` | Live combined readout in Both mode |
| `interpText(sum, secId)` | Three-band interpretation per form |
| `copySummary` | Assembles the plain-text EHR block |

## Copy-to-note output

The Copy scores button assembles a plain-text block:

```
AM-PAC "6-Clicks" Scores

Basic Mobility:
  1. Turning over in bed: 3 (A Little)
  2. Lying down to sitting on edge of bed: 3 (A Little)
  ...
  Raw score: 18/24 — 18–21 — mobile; likely home

Basic Mobility guide: 18-24 home; 17 or lower consult PT, facility possible.

Ref: Jette DU, et al. Phys Ther. 2014;94(3):379-391.
```

In Both mode the block includes both forms, both guide lines, and:

```
Combined: both >=18 home likely; either <=17 consult therapy; both <=17 facility likely.
```

## References

1. Jette DU, Stilphen M, Ranganathan VK, Passek SD, Frost FS, Jette AM. Validity of the AM-PAC "6-Clicks" inpatient daily activity and basic mobility short forms. Phys Ther. 2014;94(3):379–391. doi:[10.2522/ptj.20130199](https://doi.org/10.2522/ptj.20130199).
2. Jette DU, Stilphen M, Ranganathan VK, Passek SD, Frost FS, Jette AM. AM-PAC "6-Clicks" functional assessment scores predict acute care hospital discharge destination. Phys Ther. 2014;94(9):1252–1261. doi:[10.2522/ptj.20130359](https://doi.org/10.2522/ptj.20130359).
3. Jette DU, Stilphen M, Ranganathan VK, Passek S, Frost FS, Jette AM. Interrater reliability of AM-PAC "6-Clicks" basic mobility and daily activity short forms. Phys Ther. 2015;95(5):758–766. doi:[10.2522/ptj.20140174](https://doi.org/10.2522/ptj.20140174).
4. Howard M, Stinner D. Utilizing the AM-PAC "6-clicks" basic mobility functional assessment to determine a standard of care for the utilization of occupational therapy services status post elective spine surgery. Poster presented at: Lehigh Valley Health Network; 2021; Allentown, PA. Available from: [scholarlyworks.lvhn.org/posters/1017](https://scholarlyworks.lvhn.org/posters/1017/).
5. Martinez M, Cerasale M, Baig M, Dugan C, Robinson M, Sweis M, et al. Defining potential overutilization of physical therapy consults on hospital medicine services. J Hosp Med. 2021;16(9):553–555. doi:[10.12788/jhm.3673](https://doi.org/10.12788/jhm.3673).
6. Martinez M, Cerasale M, Baig M, Johnson JK, Dugan C, Brown A, et al. Reducing physical therapy consults for patients with high functional mobility in the acute medical inpatient setting: a difference-in-difference analysis. Arch Phys Med Rehabil. 2024;105(1):125–130. doi:[10.1016/j.apmr.2023.08.017](https://doi.org/10.1016/j.apmr.2023.08.017).
7. Boynton T. The Bedside Mobility Assessment Tool (BMAT) embedded in the new Mobility Screening and Solutions Tool (MSST): addressing inaccuracies. BMC Health Serv Res. 2024;24:1164. doi:[10.1186/s12913-024-11655-z](https://doi.org/10.1186/s12913-024-11655-z).
8. Capo-Lugo CE, McLaughlin KH, Ye B, Daley K, Young DL, Lavezza A, et al. Using nursing assessments of mobility and activity to prioritize patients most likely to need rehabilitation services. Arch Phys Med Rehabil. 2023;104(9):1402–1408. doi:[10.1016/j.apmr.2023.03.018](https://doi.org/10.1016/j.apmr.2023.03.018).

## Notes and caveats

- **Not a clinical decision system.** The interpretation guide is an informational simplification; individual patient decisions require clinical judgment.
- **Simplified cutoffs.** The 18/17 and combined-rule scheme differs from the published Basic Mobility thresholds (≤18, ≤15, ≥23); see "Relationship to the published literature" above.
- **T-score conversion.** Not implemented in this build. EHR systems convert raw AM-PAC scores to standardized T-scores using published lookup tables; the app reports raw scores only.
- **License / permissions.** The AM-PAC 6-Clicks instrument is licensed through Boston University / Pearson Assessments. This calculator reproduces the six-item Basic Mobility and Daily Activity content from the widely published clinical short forms (see refs 1–3). Institutional use should verify current licensing terms with the rights holder.

## Version

Storage schema: `ampac6clicks_calc_v4`.
