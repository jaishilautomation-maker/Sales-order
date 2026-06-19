# Pending Orders Processor

A Streamlit web app that converts raw sales-order data into a clean,
formatted, multi-sheet workbook — with no manual formatting and no
command-line usage for end-users.

## Project structure

```
pending-orders-app/
├── app.py                   # Streamlit UI
├── processor.py             # Core business logic (no Streamlit dependency)
├── requirements.txt
├── README.md
└── .streamlit/
    └── config.toml          # Theme + upload-size config
```

---

## Two input modes — auto-detected

The app figures out which mode to use from the sheet names in your files.
You don't need to select anything.

### Mode A — Single Raw Data file  (1 file)

Upload **one** `.xlsx` file that already has a sheet named **`Raw Data`**
with these columns already filled in:

| Column | Notes |
|---|---|
| `Order Type` | `Inter-depot (Jaishil)` or `External` |
| `Brand` | e.g. `Hariyali DF`, `Gain`, `NutriZin` … |
| `Depot` | e.g. `Jaipur`, `Karnal` … |
| `Customer Name` | |
| `Party Name` | |
| `SO / PO No.` | |
| `Order Date` | String format `DD-Mon-YYYY` (e.g. `15-Jun-2026`) |
| `Sched. Date` | Same format |
| `Product Name` | |
| `SKU Display` | |
| `PO Qty (unit)` | |
| `Pending Qty (unit)` | |
| `Rate (₹/unit)` | |
| `Pending (kg or ltr)` | Already-converted quantity |
| `Unit` | `kg`, `ltr`, or `pcs` |

This is the **Raw Data sheet from any previously generated output file** —
so if you ran the tool yesterday, you can re-upload yesterday's output to
regenerate today's report with updated overdue flags.

### Mode B — Two raw JSSOReport exports  (2 files)

Upload **both** raw system exports — the plant file (which contains
Sulphur Powder lines) and the Sonepat file. Each must have a sheet named
**`JSSOReport`**. The app detects which is which automatically using the
Sulphur Powder presence heuristic, then derives brand, depot, SKU, and
kg/ltr quantities from scratch.

---

## Output — 4 sheets

| Sheet | Contents |
|---|---|
| **Production Planning** | Pending qty pivoted by Brand × SKU × Depot, two blocks (WDG / Brand products) |
| **Order Queue (FIFO)** | All Jaishil inter-depot orders sorted by schedule date; overdue orders highlighted in red with ⚠ |
| **External Orders** | Sulphur Powder and Hariyali DF orders to external customers, grouped by brand |
| **Raw Data** | Full cleaned dataset — the columns are identical to the Mode A input format, so this output can be re-used as a future Mode A input |

---

## Prerequisites

- Python 3.9 or newer
- pip

---

## Installation

1. **Create and activate a virtual environment:**

   **Windows (PowerShell):**
   ```powershell
   python -m venv venv
   venv\Scripts\Activate.ps1
   ```
   If activation is blocked by execution policy, run this once first
   (in an admin PowerShell), then retry:
   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
   ```

   **macOS / Linux:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## Running

```bash
streamlit run app.py
```

Opens automatically at `http://localhost:8501`. Stop with `Ctrl+C`.

If `streamlit` isn't found on your PATH after installation:
```bash
python -m streamlit run app.py
```

---

## Usage walkthrough

1. Open the app in your browser.
2. Drag files into the upload box:
   - **1 file** → Mode A (Raw Data sheet)
   - **2 files** → Mode B (JSSOReport sheets)
   The app shows a coloured badge instantly confirming which mode was detected.
3. Click **Generate Report**.
4. Check the four metric cards (inter-depot orders, external lines, overdue count, rows processed).
5. If overdue orders exist, expand the list to see the SO numbers.
6. Click **Download Formatted Report** to save the `.xlsx`.
7. Preview the Order Queue, External Orders, and Raw Data sheets in-browser.
   Open the downloaded file in Excel to see Production Planning (merged cells
   don't render well in browser previews).

---

## Customising business rules

All rules live in `processor.py` — no other file needs to change:

| What to change | Where |
|---|---|
| SKU display name mapping | `SKU_CLEAN` dict |
| Brand detection keywords | `_get_brand()` function |
| Depot detection keywords | `_get_depot()` function |
| Depot column order in Production Planning | `DEPOT_ORDER` list |
| WDG brand order | `WDG_BRAND_ORDER` list |
| Liquid/brand order | `BRAND_BRAND_ORDER` list |
| Rate threshold for WDG qty (MT vs kg) | `MAX_RATE` constant |

After editing, restart the app (`Ctrl+C`, then `streamlit run app.py`).

---

## Deploying for your team

For colleagues who don't have Python installed, the simplest option is
**Streamlit Community Cloud** (free):

1. Push this folder to a private GitHub repository.
2. Go to [share.streamlit.io](https://share.streamlit.io) → New app → pick the repo → `app.py`.
3. Deploy — you get a shareable URL. No Python installation needed for users.

Alternatively, run it on any internal server and share
`http://<server-ip>:8501` with your team.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `streamlit: command not found` | Run `python -m streamlit run app.py` or activate your venv first |
| "doesn't contain a 'Raw Data' sheet" | You're uploading two files but one is not the Raw Data export; use either one Raw Data file or two JSSOReport files |
| "doesn't contain a 'JSSOReport' sheet" | The file was saved/modified after export; re-export from the system |
| Overdue count seems high | Check that your computer clock is set correctly; overdue is calculated vs today's date |
| WDG quantities look wrong | The `MAX_RATE = 200` threshold distinguishes MT from kg input; adjust if your rate structure has changed |
| Dates show as `NaT` in output | The Raw Data file's date strings are in an unexpected format; they should be `DD-Mon-YYYY` e.g. `15-Jun-2026` |
