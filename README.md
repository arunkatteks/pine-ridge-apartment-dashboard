# 🏢 Pine Ridge Apartment Maintenance Dashboard

A fully static, zero-dependency HTML dashboard for managing monthly apartment maintenance collection and expenses. No server required — runs entirely in the browser from a single `index.html` and `data.json`.

> **Address:** Pine Ridge Apartment, 19th Cross, KV Jairam Road, Amruthahalli (P),  
> Opp. SBI Jnana Jyothi Training Centre, Jakkuru Main Road, Bengaluru – 560092

---

## 📋 Table of Contents

- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Features](#features)
- [data.json Structure & Naming Conventions](#datajson-structure--naming-conventions)
- [Monthly Update Process](#monthly-update-process)
- [Making Changes to index.html](#making-changes-to-indexhtml)
- [Payment Details](#payment-details)
- [Running Locally](#running-locally)
- [Troubleshooting](#troubleshooting)
- [Maintainer](#maintainer)

---

## 📁 Project Structure

```
pine-ridge-dashboard/
│
├── index.html          # Main dashboard (all UI, charts, logic)
├── data.json           # All monthly maintenance & expense data
├── README.md           # This file
└── QR_CODE.jpeg        # UPI QR code (embedded in HTML as base64)
```

> ⚠️ `data.json` and `index.html` must always be in the **same folder** for the dashboard to load correctly.

---

## ⚙️ How It Works

1. When `index.html` is opened in a browser, it fetches `data.json` from the same directory.
2. The JSON is parsed and months are auto-detected by looking for sheet keys that contain the word `expenses`.
3. The month dropdown is populated automatically — newest entries appear as options.
4. Selecting a month loads the matching `Expenses <Month>` and `Maintenance <Month>` data blocks.
5. All KPI cards, charts, the apartment tracker table, and the defaulters message are rendered from that data.

---

## ✨ Features

| Feature | Description |
|---|---|
| Financial Summary | Opening balance, total collected, total expenses, pending, closing balance |
| Expense Charts | Doughnut charts by category and by description |
| Collection Status | Bar chart showing Paid / Partial / Unpaid flat counts |
| Apartment Tracker | Per-flat table with amount due, paid, balance, and colour-coded status |
| Partial Payment Highlight | Entire row turns orange when a flat has paid some but not full amount |
| Defaulters Notice | Auto-generated WhatsApp message with one-click copy |
| Pay via QR / UPI | Embedded QR code + copyable UPI ID for direct payments |
| Payment Proofs Folder | Direct link to OneDrive folder with bills and screenshots |
| Month Selector | Switch between months without reloading the page |
| Animated KPIs | Numbers count up on load for each month |

---

## 📐 data.json Structure & Naming Conventions

This is the most critical section. The dashboard reads `data.json` and matches sheet names using **exact naming patterns**. Getting these wrong will cause data to not load.

### Top-level Structure

```json
{
  "Expenses January 2025": [ ...expense rows ],
  "Maintenance January 2025": [ ...maintenance rows ],
  "Expenses February 2025": [ ...expense rows ],
  "Maintenance February 2025": [ ...maintenance rows ]
}
```

---

### ✅ Sheet Naming Convention

| Sheet Type | Pattern | Example |
|---|---|---|
| Expense sheet | `Expenses <Month> <Year>` | `Expenses March 2025` |
| Maintenance sheet | `Maintenance <Month> <Year>` | `Maintenance March 2025` |

**Rules:**
- The word `Expenses` and `Maintenance` must be spelled exactly — capital E and M, rest lowercase.
- Month must be the full English month name — `January`, not `Jan` or `01`.
- Year must be 4 digits — `2025`, not `25`.
- A space separates each word — no hyphens or underscores.
- Both sheets **must exist** for a month to display correctly. If one is missing, the other will load with empty data for the missing half.

---

### 📊 Expense Sheet — Row Format

Each row in an expense sheet must follow this structure:

```json
{
  "Description": "Lift AMC",
  "Category": "Maintenance",
  "Amount": 2500
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `Description` | String | ✅ | Free text. If it contains `opening balance` (case-insensitive), it is treated as the opening balance, not an expense. |
| `Category` | String | ✅ | Used to group expense doughnut chart. Keep consistent — see naming conventions below. |
| `Amount` | Number | ✅ | In Indian Rupees. No commas, no ₹ symbol — plain number only. |

#### Opening Balance Row (Special)

```json
{
  "Description": "Opening Balance",
  "Category": "Balance",
  "Amount": 12500
}
```

> The dashboard detects opening balance by checking if `Description` contains the text `opening balance` (case-insensitive). This row is excluded from expense totals and charts.

#### ✅ Standard Category Names (use these consistently)

Using consistent category names keeps the charts clean and comparable across months.

| Category Name | Use For |
|---|---|
| `Maintenance` | General upkeep, repairs |
| `Utilities` | Electricity, water, generator |
| `Security` | Security guard salary, agency fees |
| `Housekeeping` | Sweeper, cleaning staff salary |
| `Lift` | Lift AMC, repairs |
| `Pest Control` | Periodic pest control |
| `Miscellaneous` | Anything that doesn't fit above |
| `Balance` | Only for the Opening Balance row |

> ⚠️ Avoid variations like `maintainance`, `utility`, `misc` — these will appear as separate slices in the chart.

---

### 🏠 Maintenance Sheet — Row Format

Each row in a maintenance sheet must follow this structure:

```json
{
  "Flat": "A-101",
  "Owner": "Rajesh Kumar",
  "Amount Due": 2000,
  "Amount Paid": 2000
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `Flat` | String | ✅ | Flat number. See naming convention below. |
| `Owner` | String | ✅ | Owner or resident name. |
| `Amount Due` | Number | ✅ | Total maintenance due for the month. Usually fixed (e.g. 2000). |
| `Amount Paid` | Number | ✅ | Amount actually paid. Use `0` if unpaid. Never leave blank. |

> **Alternative field names supported:** `Flat No` instead of `Flat`, and `Owner Name` instead of `Owner` — both work.

#### ✅ Flat Number Naming Convention

| Format | Example | Notes |
|---|---|---|
| Block-Floor-Unit | `A-101` | Recommended. Block letter, floor number, unit number. |
| Block-Floor-Unit | `B-203` | Same pattern for block B |
| Numeric only | `101`, `203` | Acceptable if no block system exists |

**Rules:**
- Always use uppercase block letters: `A`, `B`, `C` — not `a`, `b`, `c`.
- Use a hyphen `-` as separator, not a slash or dot.
- Keep the format consistent across all months — the defaulters grouping groups by first character of the flat number.
- Do not include spaces in flat numbers.

#### Payment Status Logic

The dashboard determines status automatically:

| Condition | Status | Row Colour |
|---|---|---|
| `Amount Paid` = 0 | Unpaid | Red badge |
| `Amount Paid` > 0 AND < `Amount Due` | Partially Paid | **Entire row orange** |
| `Amount Paid` ≥ `Amount Due` | Paid | Green badge |

---

## 🗓️ Monthly Update Process

Follow these steps at the **start of each new month** to update the dashboard.

### Step 1 — Prepare the new month's data

Open your source spreadsheet (Excel / Google Sheets) and create two new sheets:

- `Expenses <Month> <Year>` — e.g. `Expenses June 2025`
- `Maintenance <Month> <Year>` — e.g. `Maintenance June 2025`

Fill in all rows following the field formats and naming conventions above.

---

### Step 2 — Export to JSON

Convert the spreadsheet to `data.json`. The easiest methods:

**Option A — Google Sheets (recommended)**
1. Use a Google Apps Script or a tool like [SheetDB](https://sheetdb.io) / [Sheetson](https://sheetson.com) to export sheets as JSON.
2. Alternatively, export each sheet as CSV and use [csvjson.com](https://csvjson.com/csv2json) to convert, then manually merge into the existing `data.json` structure.

**Option B — Excel**
1. Save the file as CSV per sheet.
2. Convert CSV to JSON at [csvjson.com](https://csvjson.com/csv2json).
3. Open `data.json`, add the new month's two keys and paste the arrays.

**Option C — Manual edit**
1. Open `data.json` in any text editor (VS Code recommended).
2. Add two new top-level keys following the naming convention.
3. Paste the row arrays under each key.
4. Validate the JSON at [jsonlint.com](https://jsonlint.com) before saving.

---

### Step 3 — Validate the JSON

Before pushing, always validate:

1. Go to [jsonlint.com](https://jsonlint.com).
2. Paste the full contents of `data.json`.
3. Click **Validate JSON**.
4. Fix any errors (usually a missing comma or unclosed bracket).

Common mistakes:
- Trailing comma after the last item in an array or object
- Missing quotes around field names
- Amount fields entered as strings (`"2000"`) instead of numbers (`2000`)

---

### Step 4 — Test locally

Before pushing to GitHub:

1. Open a terminal in the project folder.
2. Run a local server (see [Running Locally](#running-locally)).
3. Open the dashboard in your browser.
4. Select the new month from the dropdown.
5. Verify all KPI numbers match your spreadsheet totals.
6. Check that partial and unpaid flats show correctly in the table.

---

### Step 5 — Push to GitHub

```bash
git add data.json
git commit -m "data: add Maintenance and Expenses for June 2025"
git push
```

> Commit message convention: `data: add Maintenance and Expenses for <Month> <Year>`

---

## 🖊️ Making Changes to index.html

### When to edit index.html

- Adding a new section or card
- Updating the UPI ID or payment details
- Changing the apartment address
- Styling or colour changes
- Fixing a bug in the JavaScript logic

### Process

1. Make a backup copy: `cp index.html index.html.bak`
2. Open `index.html` in VS Code or any text editor.
3. Make your changes.
4. Test locally (see [Running Locally](#running-locally)).
5. Confirm everything works across Chrome and Safari at minimum.
6. Push to GitHub:

```bash
git add index.html
git commit -m "ui: <short description of what changed>"
git push
```

### Commit Message Conventions for index.html

| Prefix | Use When |
|---|---|
| `ui:` | Visual or layout changes |
| `fix:` | Bug fix in logic or display |
| `feat:` | New feature or section added |
| `data:` | Changes to data.json only |
| `docs:` | Changes to README only |
| `chore:` | Minor housekeeping, no user-visible change |

Examples:
```
ui: update header address text
fix: partial payment row not highlighting correctly
feat: add month-over-month trend chart
data: add Maintenance and Expenses for July 2025
```

---

### Key Areas in index.html

| What you want to change | Where to find it |
|---|---|
| UPI ID | Search for `upiIdText` — update the text content |
| Apartment address | Search for `header-address` — update the `<span>` text |
| OneDrive bills link | Search for `bills-link` — update the `href` |
| QR code image | The base64 string assigned to `QR_B64` near the `<script>` tag |
| Defaulters WhatsApp message template | Inside `renderDefaulters()` function |
| KPI card labels | Search for `class="label"` inside `.kpi-grid` |
| Chart colours | `CHART_COLORS` array in the script |

---

## 💳 Payment Details

| Detail | Value |
|---|---|
| UPI ID | `pineridgeresidentswe.62735304@hdfcbank` |
| Bank | HDFC Bank |
| Accepted apps | PhonePe, Google Pay, Paytm, BHIM, Amazon Pay, PayZapp |
| QR Code | Embedded in dashboard — scan directly from screen |

**When sharing payment confirmation, residents must include:**
1. Screenshot with date of payment clearly visible
2. Flat number (e.g. `A-101`)
3. Month the payment is for

---

## 🖥️ Running Locally

Because `index.html` fetches `data.json` via `fetch()`, opening the file directly in a browser (`file://`) may be blocked by browser security (CORS). Use a local server instead.

**Option A — Python (no install needed)**
```bash
# Python 3
python3 -m http.server 8080

# Then open: http://localhost:8080
```
MONTHLY PROCESS (Very easy now)

Every month:

Update Excel
Run:
```bash
cd "C:\Users\Arun\OneDrive\Documents\PineRidge\Tracker"

node convert.js
```

Upload updated data.json to GitHub

👉 Dashboard updates.

**Option B — Node.js**
```bash
npx serve .
# Then open the URL shown in the terminal
```

**Option C — VS Code**
Install the **Live Server** extension, right-click `index.html` → `Open with Live Server`.

---

## 🔧 Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Dashboard shows "Error loading dashboard data" | `data.json` not found or invalid JSON | Check file is in same folder; validate at jsonlint.com |
| New month not appearing in dropdown | Sheet key not matching naming convention | Check spelling — must be `Expenses Month Year` exactly |
| Flat shows wrong status | `Amount Paid` left blank instead of `0` | Set unpaid flats to `0`, never leave blank |
| Chart shows duplicate categories | Inconsistent category spelling | Standardise to the category names listed above |
| Numbers show as NaN | Amount field entered as text/string | Remove quotes around numbers in data.json |
| Page blank on double-click open | Browser blocking `fetch()` on `file://` | Use a local server as described above |

---

## 👤 Maintainer

**Designed, developed & maintained by Arun Katte**

For dashboard issues, data update queries, or feature requests — contact the maintainer or raise a GitHub Issue.

---

*Pine Ridge Apartment · Bengaluru – 560092*
