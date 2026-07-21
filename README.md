# AM-PAC "6-Clicks" Calculator

A single-file HTML calculator for the **Activity Measure for Post-Acute Care (AM-PAC) "6-Clicks"** inpatient short forms — Basic Mobility and Daily Activity / Self-Care. Bedside-ready, offline-capable, no build step, no PHI.

[![License: MIT](https://img.shields.io/badge/License-MIT-teal.svg)](LICENSE)
[![Made with HTML](https://img.shields.io/badge/single--file-HTML-blue.svg)](ampac_6clicks.html)
[![No dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](ampac_6clicks.html)

## What it does

Tap a 1–4 score for each of the six items in the Basic Mobility (PT/Nursing) or Daily Activity / Self-Care (OT/Nursing) short form. Raw totals compute live. Basic Mobility scores are interpreted against published functional thresholds for PT/OT consult necessity and discharge disposition.

## Quick start

**Option A — Open locally.** Download `ampac_6clicks.html` and open in any modern browser.

**Option B — GitHub Pages.** In your repository, enable Pages (Settings → Pages → Source: `main` / root) and browse to `https://<user>.github.io/<repo>/ampac_6clicks.html`.

**Option C — Serve.** Any static host works. No server-side code, no build tools.

```bash
# clone
git clone https://github.com/<user>/ampac-6clicks.git
cd ampac-6clicks
# open
open ampac_6clicks.html    # macOS
xdg-open ampac_6clicks.html # Linux
start ampac_6clicks.html    # Windows
```

## Features

- **Mode selector** — Basic Mobility only, Daily Activity only, or both.
- **Tap-to-score** — 4 large color-coded buttons per item (Unable / A Lot / A Little / No Assist); tap the selected score again to clear.
- **Live totals** — six-dot tracker and running raw-score chip in each section header.
- **Functional Thresholds panel** — inline PT/OT consult cutoffs (≤18, ≤15, ≥23) with clickable citation chips.
- **Score summary** — raw score /24, progress bar, and per-band interpretation.
- **Vancouver references** — 8 primary sources, ordered by first citation in the text, with clickable DOIs.
- **Copy scores** — one tap generates a plain-text score block for pasting into an EHR note.
- **Auto-save** — scores persist across reloads via `localStorage`; Reset clears both scores and mode.
- **Accessible** — radiogroup semantics, visible keyboard focus, `prefers-reduced-motion` respected.
- **Responsive** — down to phone width.
- **No PHI** — no patient name, MRN, date, or evaluator fields.

## Scoring guide

| Score | Rating        | Clinical meaning                                        |
| :---: | :------------ | :------------------------------------------------------ |
|   1   | Unable        | Total / dependent assist (>75% assistance needed)       |
|   2   | A Lot         | Max / mod assist (50–75% assistance needed)             |
|   3   | A Little      | Min / contact guard assist / supervision (<25% assist)  |
|   4   | No Assistance | Modified independent / independent (no physical help)   |

## Functional thresholds (Basic Mobility)

Hospitalized patients with Basic Mobility scores of **≤18/24** generally require PT and OT consults; scores >18 indicate high baseline mobility that rarely changes during a hospital stay. Scores of **≤15** specifically trigger OT, as they correlate with discharge to rehab facilities.

- **≤18** — primary cutoff for PT/OT necessity
- **≤15** — stronger indicator for OT consult; 14.5–17 range correlates with post-acute care / IRF discharge
- **≥23** — highly independent; rehab consult often lower-value

Published thresholds are Basic Mobility–specific. Daily Activity uses parallel descriptive bands, but no comparable published cutoffs exist.

## File structure

```
.
├── ampac_6clicks.html    # the app (single file, no deps)
├── ampac_6clicks.md      # design notes, state model, interp bands
├── README.md             # this file
├── LICENSE               # MIT (code only)
└── .gitignore
```

## Screenshots

_Add screenshots to a `docs/` folder and reference them here:_

```markdown
![Mode picker](docs/screenshot-mode.png)
![Scoring](docs/screenshot-scoring.png)
![Summary with thresholds](docs/screenshot-summary.png)
```

## Development

Nothing to install — edit `ampac_6clicks.html` directly and reload the browser. The three-file convention (HTML + Markdown design doc + optional XLSX spec) keeps documentation in lockstep with the app.

**State model:**

```js
state = {
  mode: "bm" | "da" | "both" | null,
  scores: {
    bm: [null|1|2|3|4, × 6],
    da: [null|1|2|3|4, × 6]
  }
}
```

**Persistence key:** `ampac6clicks_calc_v4` (bump the version suffix when changing the schema so old saved states don't collide).

See [`ampac_6clicks.md`](ampac_6clicks.md) for full architecture notes.

## Disclaimer

**Not medical software.** This calculator is a clinical reference tool that reproduces the arithmetic of the AM-PAC "6-Clicks" short forms and displays published threshold summaries from the peer-reviewed literature. It is **not** a medical device, is **not** FDA-cleared, and does **not** provide clinical decisions. Individual patient decisions require clinical judgment by a licensed clinician.

Cutoffs are informational summaries of the cited literature. T-score conversion is not implemented; the app reports raw scores only.

## AM-PAC instrument licensing

The AM-PAC "6-Clicks" instrument is licensed through Boston University and distributed by Pearson Assessments. This repository's **code** is MIT-licensed (see [LICENSE](LICENSE)); the six-item short-form content is reproduced from widely published peer-reviewed sources (refs 1–3 below). Institutional deployment should verify current licensing terms with the rights holder.

## References

1. Jette DU, Stilphen M, Ranganathan VK, Passek SD, Frost FS, Jette AM. Validity of the AM-PAC "6-Clicks" inpatient daily activity and basic mobility short forms. *Phys Ther.* 2014;94(3):379–391. doi:[10.2522/ptj.20130199](https://doi.org/10.2522/ptj.20130199).
2. Jette DU, Stilphen M, Ranganathan VK, Passek SD, Frost FS, Jette AM. AM-PAC "6-Clicks" functional assessment scores predict acute care hospital discharge destination. *Phys Ther.* 2014;94(9):1252–1261. doi:[10.2522/ptj.20130359](https://doi.org/10.2522/ptj.20130359).
3. Jette DU, Stilphen M, Ranganathan VK, Passek S, Frost FS, Jette AM. Interrater reliability of AM-PAC "6-Clicks" basic mobility and daily activity short forms. *Phys Ther.* 2015;95(5):758–766. doi:[10.2522/ptj.20140174](https://doi.org/10.2522/ptj.20140174).
4. Howard M, Stinner D. Utilizing the AM-PAC "6-clicks" basic mobility functional assessment to determine a standard of care for the utilization of occupational therapy services status post elective spine surgery. Poster presented at: Lehigh Valley Health Network; 2021; Allentown, PA. Available from: [scholarlyworks.lvhn.org/posters/1017](https://scholarlyworks.lvhn.org/posters/1017/).
5. Martinez M, Cerasale M, Baig M, Dugan C, Robinson M, Sweis M, et al. Defining potential overutilization of physical therapy consults on hospital medicine services. *J Hosp Med.* 2021;16(9):553–555. doi:[10.12788/jhm.3673](https://doi.org/10.12788/jhm.3673).
6. Martinez M, Cerasale M, Baig M, Johnson JK, Dugan C, Brown A, et al. Reducing physical therapy consults for patients with high functional mobility in the acute medical inpatient setting: a difference-in-difference analysis. *Arch Phys Med Rehabil.* 2024;105(1):125–130. doi:[10.1016/j.apmr.2023.08.017](https://doi.org/10.1016/j.apmr.2023.08.017).
7. Boynton T. The Bedside Mobility Assessment Tool (BMAT) embedded in the new Mobility Screening and Solutions Tool (MSST): addressing inaccuracies. *BMC Health Serv Res.* 2024;24:1164. doi:[10.1186/s12913-024-11655-z](https://doi.org/10.1186/s12913-024-11655-z).
8. Capo-Lugo CE, McLaughlin KH, Ye B, Daley K, Young DL, Lavezza A, et al. Using nursing assessments of mobility and activity to prioritize patients most likely to need rehabilitation services. *Arch Phys Med Rehabil.* 2023;104(9):1402–1408. doi:[10.1016/j.apmr.2023.03.018](https://doi.org/10.1016/j.apmr.2023.03.018).

## License

Code is released under the [MIT License](LICENSE). The AM-PAC instrument itself is separately licensed — see the AM-PAC Instrument Licensing note above.
