# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A self-contained static HTML page (`index.html`) that visualizes all fields in EU eForms 16 (Contract Notice) and 29 (Contract Award Notice). It shows field requirements (M/CM/EM/O), data types, and dropdown values from the EU and Norwegian eForms SDKs. Intended as a reference tool for PMs, business analysts, and developers working on the SourceMagnet Doffin integration.

## Architecture

This repo only contains the **built output** (`index.html`). The build tooling lives in the parent directory `/Users/marina/bootcamp/`:

- `build-eforms-overview.py` — Python script that generates `index.html`
- `eforms-overview-template.html` — HTML/CSS/JS template
- `sdk-reference/` — Norwegian SDK YAML codelists + BT→codelist mapping JSONs
- `20250121_eforms_3rd_amendment.xlsx` — EU regulation field matrix (source of truth for field requirements)

The build script reads the Excel + Norwegian YAML codelists + fetches ~60 EU SDK Genericode files from GitHub, then embeds everything as JSON into the HTML template.

## Regenerating the HTML

```bash
cd /Users/marina/bootcamp
python3 build-eforms-overview.py
```

Requirements: `pandas`, `openpyxl`, `pyyaml` (Python packages). Fetches EU SDK codelists from `github.com/OP-TED/eForms-SDK/1.13.2/codelists/` at build time.

## Deploying Updates

Hosted on Cloudflare Workers at `https://eforms-overview.marina-254.workers.dev/`

```bash
cp /Users/marina/bootcamp/eforms-wizard-overview.html /Users/marina/bootcamp/eforms-overview/index.html
cd /Users/marina/bootcamp/eforms-overview
git add index.html
git commit -m "Update overview"
git push
```

Auto-deploys on push (~30 seconds).

## Data Sources

- **Field definitions**: `20250121_eforms_3rd_amendment.xlsx`, sheet `2023_2884_3rd_amendment`, columns X (F16) and AL (F29)
- **Norwegian SDK 1.13.2**: `sdk-reference/codelists/*.yaml` — copied from `sm-webapp-bff` branch `feat/SM-136-doffin-eform-16-integration`
- **EU SDK 1.13.2**: Fetched at build time from `github.com/OP-TED/eForms-SDK`
- **BT→codelist mapping**: `sdk-reference/bt_codelist_map.json` (extracted from EU SDK `fields.json`)
- **Codelist→filename mapping**: `sdk-reference/codelist_filenames.json` (extracted from EU SDK `codelists.json`)

## Testing

Data integrity tests verify the HTML output matches the Excel source of truth:

```bash
cd /Users/marina/bootcamp
python3 -m pytest test_eforms_overview.py -v
```

25 tests covering: field counts, Excel comparison (row-by-row), wizard step assignment, codelist mappings, Norwegian SDK values, BFF mapping status, and HTML output sanity. Run after regenerating to catch data mismatches before deploying.

## Gotchas

- Build requires internet — fetches ~60 EU SDK `.gc` files from GitHub at build time
- Excel column positions are hardcoded: F16 = col index 23 (X), F29 = col index 37 (AL). If the Excel structure changes, update `extract_excel_fields()` in the build script
- BT-799 appears twice in the Excel (row 50 under BG-715, row 362 under BG-716) — both are extracted, the second has the actual F29=CM value
- 3 code fields (BT-08, BT-770, BT-814) were manually added to `bt_codelist_map.json` — not in the EU SDK `fields.json` under those BT IDs
- The `index.html` in this repo is the **generated output** — never edit it directly, edit the template and rebuild

## Key Files to Edit

- **Template changes** (layout, CSS, JS filters): edit `eforms-overview-template.html` in `/Users/marina/bootcamp/`, then rebuild
- **Data/field changes**: update the Excel or SDK reference files, then rebuild
- **Wizard step grouping**: defined in `WIZARD_STEPS` array in `build-eforms-overview.py`
