# AM-PAC "6-Clicks" Calculator

Single-file HTML calculator for the Activity Measure for Post-Acute Care "6-Clicks" inpatient short forms. Bedside-ready, offline-capable, and self-contained — no build step, no dependencies, no PHI storage.

## What it does

Lets a clinician tap a 1–4 score for each item of the Basic Mobility (PT/Nursing) or Daily Activity / Self-Care (OT/Nursing) short form and computes raw totals automatically. The user chooses which short form(s) to score at the top; only the chosen section(s) render. Basic Mobility raw scores are interpreted against the published functional thresholds for PT/OT consult necessity and discharge disposition.

## Features

- **Mode selector.** Basic Mobility only, Daily Activity only, or Both.
- **Tap-to-score UI.** 4 large color-coded buttons (Unable / A Lot / A Little / No Assist) per item; tap a selected score again to clear.
- **Live totals.** Six-dot progress tracker and running raw score chip in each section header; chip turns teal when all 6 items are scored.
- **Score summary card.** Raw score /24, progress bar, and per-band interpretation.
- **Functional Thresholds panel** (Basic Mobility only) with the published PT/OT consult cutoffs and their supporting citations as clickable chips.
- **Vancouver-format references** with clickable DOIs, numbered by order of first citation in the text.
- **Copy scores.** Generates a plain-text score block for pasting into an EHR note, including the Basic Mobility cutoffs when BM is in scope.
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

### Basic Mobility (PT / Nursing)

1. Turning over in bed
2. Lying down to sitting on edge of bed
3. Bed to chair (seated transfer)
4. Standing up from a chair
5. Walking in room
6. Climbing 3–5 steps with a railing

### Daily Activity / Self-Care (OT / Nursing)

1. Feeding / eating meals
2. Personal grooming (teeth, face)
3. Upper body dressing (on & off)
4. Lower body dressing (on & off)
5. Bathing (washing, rinsing, drying)
6. Toileting (toilet, bedpan, or urinal)

## Functional thresholds (Basic Mobility)

Hospitalized patients with AM-PAC "6-Clicks" Basic Mobility scores of **≤18/24** generally require PT and OT consults, as scores >18 indicate high baseline mobility that rarely changes during a hospital stay. Scores of **≤15** specifically trigger OT, as they correlate with discharge to rehab facilities.<sup>4,5,6</sup>

- **≤18 Basic Mobility** — low mobility and the primary cutoff for PT/OT necessity. Patients scoring above this are often over-consulted, as they can usually be discharged home without significant functional changes.<sup>5,6,7</sup>
- **≤15 Basic Mobility** — stronger indicator for OT consult; patients scoring in the 14.5–17 range frequently require post-acute care or inpatient rehabilitation.<sup>4</sup>
- **≥23** — highly independent function. Helps nursing staff and hospitalists identify patients who may not benefit from rehab services (consults often lower-value or limited to a single visit).<sup>8,4</sup>

Published thresholds are Basic Mobility–specific; parallel descriptive bands are shown for Daily Activity, but no comparable Daily-Activity cutoffs have been published.

### Interpretation bands used in the summary card

**Basic Mobility**

| Score | Band                                                                                    |
| :---: | :-------------------------------------------------------------------------------------- |
| ≥23   | Highly independent; rehab consult often lower-value                                     |
| 19–22 | Higher baseline mobility; PT/OT rarely change trajectory                                |
| 16–18 | Low mobility; PT/OT consult typically indicated                                         |
| 12–15 | Very low mobility; OT consult indicated; higher likelihood of post-acute care           |
| ≤11   | Severe impairment; dependent range                                                      |

**Daily Activity** (descriptive; no published cutoffs)

| Score | Band                                          |
| :---: | :-------------------------------------------- |
| ≥23   | Largely independent in self-care              |
| 19–22 | Mild impairment in self-care                  |
| 16–18 | Moderate impairment; assistance needed        |
| 12–15 | Substantial dependence in self-care           |
| ≤11   | Severe impairment; dependent range            |

## Architecture

Single-file HTML with inline `<style>` and `<script>`. No external assets, no CDN calls, no network dependencies. Governed by the same three-file lockstep convention used elsewhere:

| File                    | Purpose                                              |
| :---------------------- | :--------------------------------------------------- |
| `ampac_6clicks.html`    | The app itself.                                      |
| `ampac_6clicks.md`      | This document (design notes + citations).            |
| `ampac_6clicks.xlsx`\*  | Reserved for a spec workbook if formalized later.    |

\*Not currently created.

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

Persisted to `localStorage` under the key `ampac6clicks_calc_v4`. Reset clears both the persisted state and the DOM.

### Scoring math

Raw score = sum of six items (min 6, max 24). Section shown as `— / 24` until all six items are scored; if fewer than six are complete, a partial sum is displayed with an asterisk and the count of items scored. Progress-bar fill = `((raw − 6) / 18) × 100%`.

## Copy-to-note output

The Copy scores button assembles a plain-text block:

```
AM-PAC "6-Clicks" Scores

Basic Mobility:
  1. Turning over in bed: 3 (A Little)
  2. Lying down to sitting on edge of bed: 3 (A Little)
  ...
  Raw score: 18/24 — 16–18 — low mobility; PT/OT consult typically indicated

Basic Mobility cutoffs: ≤18 low mobility (PT/OT indicated);
  ≤15 OT consult / post-acute care likely; ≥23 highly independent.

Ref: Jette DU, et al. Phys Ther. 2014;94(3):379-391.
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

- **Not a clinical decision system.** Cutoffs are informational summaries of published thresholds; individual patient decisions require clinical judgment.
- **T-score conversion.** Not implemented in this build. EHR systems convert raw AM-PAC scores to standardized T-scores using published lookup tables; the app reports raw scores only.
- **License / permissions.** The AM-PAC 6-Clicks instrument is licensed through Boston University / Pearson Assessments. This calculator reproduces the six-item Basic Mobility and Daily Activity content from the widely published clinical short forms (see refs 1–3). Institutional use should verify current licensing terms with the rights holder.

## Version

Storage schema: `ampac6clicks_calc_v4`.
