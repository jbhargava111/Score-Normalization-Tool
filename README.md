# [MIT $100K](https://www.mit100k.org/) Score Normalizer

A single-file, browser-based tool that normalizes judge scores for the MIT $100K Entrepreneurship Competition. Paste in the raw evaluation export from your judging platform and get a ranked list of applications that corrects for judges who score harsher or more generously than their peers.

No install, no server, no login. Everything runs locally in the browser, and no data ever leaves your machine.

---

## Why this exists

Not every application is reviewed by the same judges, and not every judge scores the same way. Some applications get one reviewer, others get 20+. Some judges run harsh, others run generous. A raw average rewards whichever applications happened to draw lenient judges.

Score normalization corrects for each judge's personal scoring tendency (their own average and spread) before comparing applications, so rankings reflect the venture rather than the luck of the draw.

This tool replaces a multi-tab Excel workbook that required manual pivots, copy-pasted reviewer counts, and hand-cleaned data.

---

## Quick start

1. Download `score-normalizer.html`.
2. Double-click it to open in any modern browser (Chrome, Safari, Firefox, Edge).
3. Paste your raw evaluation export into the box and follow the three steps on screen.

That's it. The file works offline once downloaded.

---

## How to use it

### Step 1: Paste your data
Copy the evaluation export straight from Excel or your platform (tab-separated works best, comma-separated also works) and paste it in, **including the header row**.

The export should have one row per judge-application review. At minimum it needs a judge, an application, and a total score. Example:

| Submission Name | Submission ID | Judge Name | Total | Finished Evaluating |
|---|---|---|---|---|
| Acme Robotics | 9001234 | Jane Doe | 21 | yes |
| Acme Robotics | 9001234 | John Smith | 16 | yes |
| Beta Health | 9005678 | Jane Doe | 18 | yes |

Extra columns (comments, emails, timestamps) are fine. Subtotal rows and blank spacer rows in the export are skipped automatically.

### Step 2: Map your columns
The tool auto-detects columns by header name. Always glance over the mapping before computing, especially with a new export format.

| Role | Required? | Purpose |
|---|---|---|
| Judge | Yes | Who scored |
| Application | Yes | What was scored |
| Application ID | Recommended | The unique identifier. Applications are grouped by ID, so name typos never split one application in two |
| Total score | Yes (or map category scores to sum) | The basis for the official ranking |
| Finished Evaluating | Recommended | Lets you exclude unfinished reviews |
| Category 1/2/3 scores | Optional | Diagnostic breakdown in the drill-down only |
| Category 1/2/3 comments | Optional | Readable in the drill-down via double-click |

Two checkboxes control data cleaning. **Keep both on** (they are on by default):

- **Exclude unfinished:** drops rows not marked as finished.
- **Remove exact duplicate rows:** drops accidental repeat entries (same judge, same application, identical score) before computing.

### Step 3: Review results
Applications are ranked by **Normalized score**. **Standard score** (the plain average of raw totals) is shown alongside purely for comparison. Click any application to expand it and see every individual review.

Other controls:

- **Finalist cutoff (top N):** flags applications whose finalist status changes depending on Normalized vs. Standard scoring.
- **Reviewer threshold:** flags applications with fewer than N reviewers (default 3). These are still ranked, just flagged.
- **Export CSV:** downloads the ranked results.
- **Edit mapping / Edit data / Start over:** go back without losing your place.

---

## Methodology

All normalization is based on each review's **Total** score.

**1. Judge calibration.** For each judge, compute the mean and sample standard deviation (n-1) of all their Total scores across every application they reviewed.

**2. Z-score.** Convert each review to a z-score relative to that judge:

```
z = (Total - judge mean) / judge standard deviation
```

A judge with only one review, or whose scores show zero spread, has no meaningful personal calibration, so their z-score is set to 0.

**3. Rescale per application.** For each application, take the minimum and maximum raw Total and the minimum and maximum z-score across its reviewers, then:

```
avgRaw = (minRaw + maxRaw) / 2
avgZ   = (minZ + maxZ) / 2
slope  = (maxRaw - minRaw) / (maxZ - minZ)

normalized value (per review) = avgRaw + slope * (z - avgZ)
```

The application's **Normalized score** is the mean of its per-review normalized values. This keeps scores on the familiar raw-score scale while correcting for judge severity.

If an application has only one review, or every reviewer has an identical z-score, there is nothing to rescale against, and the raw Total is used.

**4. Standard score.** The plain average of raw Totals, with no adjustment.

---

## Flags

Review every flag before trusting a ranking. Do not just read the top-line score.

| Flag | Meaning | What to do |
|---|---|---|
| **outlier** | A review's raw total is more than 2 standard deviations from the mean of that application's other included reviews (requires 3+ reviews) | Double-click the judge to read their comments. Decide whether it is a legitimate score or a mismatch, and use **Exclude** if appropriate |
| **conflict — averaged** | The same judge scored the same application more than once with different totals | The scores were averaged into one entry. Double-click the judge's name to see the originals |
| **duplicate row** | An exact repeat entry was found | Removed or kept depending on the checkbox. Confirm it matches what you expect |
| **cutoff-sensitive** | Finalist status changes between Normalized and Standard scoring | These are the true borderline calls. Compare both scores directly |
| **N reviewers** | Fewer reviewers than the threshold | Still fully ranked. Treat with lower confidence |
| **all reviews excluded** | Every review for an application was manually excluded | Re-include at least one to restore its ranking |

### Manual exclusion
Every review in the drill-down has an **Exclude / Re-include** button. Excluded reviews stay visible (greyed out) so the decision is always reversible, but they are removed from every calculation, including each judge's own mean and standard deviation.

### Reading comments
If comment columns are mapped, any cell with a comment shows a hint in the drill-down:

- **Double-click a judge's name** to see all of their comments (and original scores for conflict-averaged entries).
- **Double-click a score cell** to see the comment for that single category.

---

## Hosting for your team

The tool is one static HTML file, so it can live anywhere:

- **Shared drive:** upload `score-normalizer.html` and have teammates download and open it. Note that Google Drive previews HTML as plain text rather than running it.
- **GitHub Pages:** enable Pages on this repository (Settings → Pages → deploy from branch) and the tool will be available at a stable URL with no changes needed.

---

## Data privacy

- Everything runs client-side. Pasted data is held in memory only and is never uploaded, stored, or transmitted.
- Closing or refreshing the tab clears everything.
- **Do not commit real judging data** (exports, rankings, judge names, or comments) to this repository, especially if it is public.

---

## Repository contents

```
score-normalizer.html   # the entire tool: UI, logic, and styling in one file
README.md
```

## Browser support

Any modern evergreen browser. Fonts load from Google Fonts when online and fall back to system fonts offline.

## Contributing / handoff notes

Because the whole tool is a single file, changes are easy to review and hand off between organizing teams. If you modify the methodology, update the Methodology section above and re-validate against a known cycle's output before relying on new results.

---

