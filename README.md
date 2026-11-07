# Job Sweeper — Full Sweep Reference

All sweep methods, flags, and workflows in one place.

**Profile:** Asal Sepehri — kundservice, butik/kassa, administration, kriminalvård, stöd & skönhet (Stockholms & Uppsala län). Keywords come from the CV (`Asal_Sepehri_CV.docx` / `Asal_SepehriCV.pdf`), the personligt brev and the certificates/transcripts (Kriminalvården e-kurser, Swedavia Safety & Security, Safe@Work, Burgården Bar & Servering, Komvux, VHS gymnasiebedömning, Dr. Belter, ABSOLUT, Ladok: Borås / Sophiahemmet / Umeå HR); see `job-search-keywords-all-comprehensive.txt`.

> Keyword sets live in `job-search.js` (CORE_ROLE_TERMS, SECURITY_ANALYSIS_TERMS, SECTOR_KEYWORD_TERMS feed the search queries; SECTOR_CERT_TERMS and EMPLOYER_NAME_TERMS are scoring/matching only). Edit the lists there directly when the CV changes.

**Folder:** the folder this README sits in (run all commands from here)

## Contents

| Phase | What | Type |
|---|---|---|
| [1](#phase-1--automated-sweep-automatic) | Automated sweep (Platsbanken, Bing, LinkedIn, Jobbsafari, municipalities) | `[automatic]` |
| [2](#phase-2--google-sweep-manual) | Google sweep via VS Code extension | `[manual]` |
| [3](#phase-3--indeed-sweep-browser-required) | Indeed sweep via Edge/CDP | `[browser required]` |
| [4](#phase-4--nystartsjobb-search-separate) | Nystartsjobb search (all pages, Stockholm) + Indeed nystart queries | `[separate]` |
| [5](#phase-5--after-the-sweep-every-time) | Filter + log new leads | `[every time]` |

---

## Phase 1 — Automated sweep `[automatic]`
> Covers Platsbanken, Bing RSS, LinkedIn public feed, Jobbsafari, municipality feeds. No browser needed.

### Full sweep (all API sources)
```
node job-search.js
```

### Platsbanken only (quickest daily check)
```
node job-search.js --platsbanken-only
```

> Results auto-save as timestamped `.txt` files, pre-filtered against your exclusion lists.

---

## Phase 2 — Google sweep `[manual]`
> Node.js is blocked by Google. Use the VS Code extension + Chrome DevTools to extract results manually.

### Step 1 — Open the panel in VS Code
```
Ctrl+Shift+P  →  jobSweeper.sweepGoogle
```

### Step 2 — Extract results from Chrome
1. Click **COPY SCRIPT** in the panel
2. Go to a Google job search results page in Chrome
3. Open DevTools: `F12` → Console tab
4. Paste the script and press Enter
5. Console returns `undefined` — correct, JSON is now in your clipboard

> **Tip:** Save the script as a Chrome bookmarklet for one-click extraction without opening DevTools.

### Step 3 — Paste into panel and save
Back in VS Code: click the textarea → `Ctrl+V` → click **SAVE**.

Results write to:
```
google-sweep-YYYY-MM-DD.txt
```

### Step 4 — Advance to next query
Click **NEXT** in the panel. It builds the next Google URL automatically. Open in Chrome, run the script, paste back. Repeat until all queries are done.

---

## Phase 3 — Indeed sweep `[browser required]`
> Standard `job-search.js` always returns 0 from Indeed due to anti-bot blocking. Always use the method below.

### ⚠️ Must run from VS Code terminal — CAPTCHA only clears in VS Code browser context

### Step 1 — Launch Edge with remote debugging
```
.\Launch-EdgeForSweep.ps1
```
Opens Edge on port 9222 using your saved profile. Solve any CAPTCHA manually in the Edge window.

> **Port 9222 is local only** — VPN changes do not affect it. You do not need to update anything when switching VPN.

### Step 2 — Run the sweep (from VS Code terminal)
```
node indeed-capture-sweep.js
```

Playwright connects to the already-open Edge window via CDP. No new browser launched, no conflict.

**Output:** `indeed-browser-jobs-vscode-total.json` (merges with previous results — never overwrites)

### Command reference

**Basics**
```
node indeed-capture-sweep.js
node indeed-capture-sweep.js --results relevance --pages 2 --out indeed-relevance.json
node indeed-capture-sweep.js --results date --fresh        # ignore saved progress, start over
node indeed-capture-sweep.js --start-at 25                 # begin at query #25
```

**Relevance, starting at query #25**
```
node indeed-capture-sweep.js --results relevance --start-at 25
```

**Date (newest first), starting at query #25**
```
node indeed-capture-sweep.js --results date --start-at 25
```

**Relevance, 2 pages**
```
node indeed-capture-sweep.js --results relevance --pages 2
```

**Date (newest first), 2 pages**
```
node indeed-capture-sweep.js --results date --pages 2
```

**Start at query #25**
```
node indeed-capture-sweep.js --results relevance --pages 2 --start-at 25
node indeed-capture-sweep.js --results date --pages 2 --start-at 25
```

**Separate output files**
```
node indeed-capture-sweep.js --results relevance --pages 2 --out indeed-relevance.json
node indeed-capture-sweep.js --results date --pages 2 --out indeed-date.json
```

**Start over, ignoring saved progress**
```
node indeed-capture-sweep.js --results date --pages 2 --fresh
```

### Custom output file
```
node indeed-capture-sweep.js --out my-results.json
```

### Method B — VS Code extension (fallback)
```
Ctrl+Shift+P  →  jobSweeper.sweepIndeed
```
Follow the same COPY SCRIPT → Chrome DevTools → paste → SAVE flow as Google.

---

## Phase 4 — Nystartsjobb search `[separate]`
> Kept completely apart from the normal sweep. Searches Platsbanken for `nystartsjobb`, walks **every result page**, and keeps **Stockholms län** ads only.

**Search link:** https://arbetsformedlingen.se/platsbanken/annonser?q=nystartsjobb

### Run the nystart search
```
node job-search.js --nystart
```

| Variant | What it does |
|---|---|
| `node job-search.js --nystart` | All pages, Stockholm ads only, CV-relevance filter on. |
| `node job-search.js --nystart --nystart-all` | Same, but skip the CV-relevance filter. |
| `node job-search.js --nystart --nystart-sweden` | Keep ads from every län, not just Stockholm. |
| `node job-search.js --nystart --posted-within r1209600` | Only ads posted in the last 14 days. |
| `node job-search.js --nystart --export nystart-today.txt` | Write results to a custom file. |

**Output files**
- `nystart-results.txt` — the ads, in the normal `[NEW]` block format (pastes into `asal-links-poster.html`).
- `nystart-results-pages.txt` — full page index: page number, offset, ads per page, Stockholm ads per page, API URL and ad links.

> The JobTech API stops serving results at offset 2000. If the full list is larger, the script automatically re-runs län by län so no page is missed.

### Nystart queries on Indeed
`indeed-capture-sweep.js` also includes the same three terms, run for Stockholms län and Uppsala län:

- `nystartsjobb`
- `nystartsjobben`
- `nystartsanställning`

With the default alphabetical sort they are queries #74–76 (of 125). To jump straight to them (it then continues through the remaining queries):
```
node indeed-capture-sweep.js --start-at 74
```

---

## Phase 5 — After the sweep `[every time]`

### 1 — Filter against `jobs_list.pdf` (222 entries)
Any result that appears here has already been processed. Skip it.

### 2 — Filter against `Total_Jobs.pdf` (1004 entries)
Any match here has been seen. Skip it. Remaining results are genuinely new.

### 3 — Log new leads in `Jobs_applied.xlsx`
New jobs that pass both filters → **Jobs new CV** or **Active CV Tests** sheet depending on status.

---

## All flags

| Flag | What it does |
|---|---|
| `--platsbanken-only` | Query only Platsbanken API. Fastest daily check. |
| `--indeed-browser-capture` | Open Playwright browser for Indeed — solve CAPTCHA manually. |
| `--indeed-browser-assisted` | Use browser-assisted mode for Indeed scraping. |
| `--indeed-browser-use-snapshot` | Reuse a saved browser snapshot. Use with `--indeed-browser-assisted`. |
| `--no-filter` | Skip exclusion list filtering — see all raw results. |
| `--out <filename>` | Write Indeed results to a custom file instead of default. |
| `--nystart` | Separate nystartsjobb search (all pages, Stockholm). See Phase 4. |
| `--nystart-all` | With `--nystart`: skip the CV-relevance filter. |
| `--nystart-sweden` | With `--nystart`: keep ads from every län. |

Flags can be combined:
```
node job-search.js --platsbanken-only --no-filter 

node job-search.js --posted-within=r5184000 --no-bing

node job-search.js --platsbanken-only --nystart
```

---

## Maintenance scripts

| Script | What it does |
|---|---|
| `.\Launch-EdgeForSweep.ps1` | Launches Edge on port 9222 for Indeed sweep. Run before `node indeed-capture-sweep.js`. |

---

## Source status

| Source | Method | Status | Notes |
|---|---|---|---|
| Platsbanken | `job-search.js` | ✅ Working | Most reliable. Use `--platsbanken-only` for quick checks. |
| Platsbanken nystart | `node job-search.js --nystart` | ✅ Working | Separate search, all pages, Stockholm only. |
| Bing RSS | `job-search.js` | ✅ Working | Included in default full sweep. |
| LinkedIn | `job-search.js` | ✅ Working | Public feed only, no auth required. |
| Jobbsafari | `job-search.js` | ✅ Working | Included in default sweep. |
| Municipality feeds | `job-search.js` | ✅ Working | Stockholm/Uppsala region direct boards. |
| Google Search | VS Code extension | ⚠️ Manual only | Node.js blocked. Use `jobSweeper.sweepGoogle` + DevTools script. |
| Indeed | Playwright via CDP | ✅ Working | Run `Launch-EdgeForSweep.ps1` first, then `node indeed-capture-sweep.js`. Includes nystart queries. |
| Säkerhetsvarlden | `node job-search.js --sakerhetsvarlden` | ⏸ Off by default | Security-industry source — not relevant to this profile. |

---

## Troubleshooting

### Edge won't connect / CAPTCHA loop
Run from **VS Code terminal only** — not from a standalone PowerShell window. The CAPTCHA session is tied to the VS Code browser context.

### "Existing session" / browser conflict error
Close all open Edge/Chrome windows, then run `.\Launch-EdgeForSweep.ps1` again.

### Port 9222 already in use
The launcher script handles this automatically. If it still fails:
```powershell
Stop-Process -Name "msedge" -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 2
.\Launch-EdgeForSweep.ps1
```

### ERR_INVALID_PACKAGE_CONFIG
`package.json` has been corrupted. Restore it:
```powershell
'{"name":"keywords-sweep","version":"1.0.0","main":"job-search.js"}' | Out-File -Encoding utf8 package.json
```

### Extension command not appearing in Ctrl+Shift+P
Extension host failed to load. Check VS Code Developer Tools (`Help → Toggle Developer Tools`) for errors referencing `job-sweeper`. If corrupt, re-run the `write-ext.js` fix from Downloads.

### Google extraction returns empty array
Google changed their HTML selectors. The working selector is `[data-hveid] h3`. If results stop coming back, check that the extraction script uses this selector.

### Indeed sweep shows 0 new results
The output file (`indeed-browser-jobs-vscode-total.json`) already contains those jobs from a previous run — this is expected. Use `--out newfile.json` to see raw unfiltered results.

### Nystart search returns no ads
Open the search link in the browser to see the live Platsbanken list. If the API is unreachable, re-run later — each page is retried up to 3 times before it is flagged `FETCH FAILED` in `nystart-results-pages.txt`.
#   a s a l  
 #   a s a l  
 