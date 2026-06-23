# HashHarvest — Usage Guide

HashHarvest (v0.7.0) is a desktop GUI tool for extracting cryptographic hashes (MD5, SHA1, SHA256, SHA512) from files. It is designed for incident response, forensics, and threat intelligence workflows where you need to find, triage, and export hash evidence quickly.

---

## Table of Contents

1. [Running from Source](#running-from-source)
2. [Running from the Executable](#running-from-the-executable)
3. [Running a Scan](#running-a-scan)
4. [Results Table](#results-table)
5. [Watchlist](#watchlist)
6. [Scan History](#scan-history)
7. [Exporting Results](#exporting-results)

---

## Running from Source

### Prerequisites

- Python 3.9 or later
- pip

### Setup

```bash
git clone https://github.com/labgeek/HashHarvest.git
cd HashHarvest
pip install -r requirements.txt
```

### Launch

```bash
python -m hashharvest.main
```

> Run it as a module with `-m` from the project root. Running the file directly (`python hashharvest/main.py`) will fail because the package uses absolute imports that require the project root on `sys.path`.

---

## Running from the Executable

Download the latest `HashHarvest.exe` from the [Releases](https://github.com/labgeek/HashHarvest/releases) page, or build it yourself:

### Build from Source

Requires PyInstaller:

```bash
pip install pyinstaller
python -m PyInstaller --clean --onefile --windowed --name HashHarvest --hidden-import PyQt5.sip hashharvest/main.py
```

The executable is written to `dist\HashHarvest.exe`.

### Launch

Double-click `HashHarvest.exe` — no Python installation required.

> The `hashharvest.db` scan database is created in the same folder as the executable on first run.

---

## Running a Scan

### 1. Select an Input Directory

Click **Select Input Folder** and choose the directory containing the files you want to scan. HashHarvest walks the directory recursively and processes every supported file it finds.

**Supported file types:** PDF, TXT, LOG, MD, CSV, JSON, XML, DOCX, XLSX, PPTX

> Microsoft Office files (`.docx`, `.xlsx`, `.pptx`) are parsed directly from their OpenXML contents — no Office installation or extra libraries required. Text split across runs within a paragraph or cell is reassembled before matching. Word reads the document body; Excel reads string cells across all worksheets; PowerPoint reads slide text.

### 2. Choose Hash Types

Check or uncheck **MD5**, **SHA1**, **SHA256**, and **SHA512** to control which algorithms to scan for. All four are selected by default. Deselect algorithms you don't need to speed up large scans.

### 3. Run the Scan

Click **Start Scan**. The progress bar updates as each file is processed. Files that cannot be read are skipped and counted under **Skipped Files** — the scan always continues to completion.

Results appear in the table as they are found, in real time.

### 4. Clear and Re-scan

Click **Clear Form** to reset all fields, results, progress, and summary counts before starting a new scan.

---

## Results Table

The results table has six columns:

| Column | Description |
|--------|-------------|
| **Source File** | Absolute path to the file containing the hash. Long paths are middle-elided; hover for the full path tooltip. |
| **File Type** | File extension in uppercase (PDF, LOG, DOCX, etc.). |
| **Hash Type** | Algorithm: MD5, SHA1, SHA256, or SHA512. |
| **Hash Value** | Lowercase hexadecimal hash string. Stretches to fill the window by default. |
| **Line** | Line number within the file where the hash first appears. Hidden by default. |
| **Context** | Up to 60 characters on either side of the match, with newlines collapsed to spaces. PDF and PPTX hits include a `[page N]` or `[slide N]` prefix. Stretches to fill width when visible. |

Only the first occurrence of each (algorithm, value) pair per file is shown — a hash that appears multiple times on different lines is recorded once, at its first position.

### Showing Line and Context

Check **Show Context** above the table to reveal the **Line** and **Context** columns. Uncheck to hide them again. The column that fills available horizontal space switches automatically between Hash Value and Context so the table always fills the window cleanly.

### Filtering Results

Type any text into the filter bar above the table to hide rows that do not match. The filter is applied across all columns simultaneously — you can filter by file name, hash type, hash value fragment, or any context text. The filter clears automatically at the start of each new scan.

### Sorting

Click any column header to sort by that column. Click again to reverse the sort. Sorting is re-enabled automatically after each scan completes.

### Right-click Actions

Right-click any row for quick copy actions:

| Action | Copies to clipboard |
|--------|-------------------|
| **Copy Hash** | The hash value from the selected row. |
| **Copy Row** | All visible columns of the selected row, tab-separated. |

---

## Watchlist

The Watchlist is the triage layer on top of extraction. Load known-bad hash lists before or after a scan and any matching rows are immediately highlighted red.

### Opening the Watchlist Manager

Click **Watchlist** in the toolbar to open the Watchlist Manager.

### Creating a Watchlist

Click **New…**, type a name (e.g. `Incident 2026-06`, `IOC Feed`), and press OK. You can maintain multiple named watchlists simultaneously — all active lists are checked after every scan.

### Importing Hashes

1. Select a watchlist from the list on the left.
2. **Paste method** — paste any text containing hashes into the text area, then click **Import from Text**.
3. **File method** — click **Browse File…** and select a `.txt`, `.csv`, or `.log` file; hashes are extracted and imported immediately without filling the text area.

The importer finds any valid MD5 / SHA1 / SHA256 / SHA512 hex string in the input and discards everything else. You can paste raw hash lists, threat-intel CSV exports, STIX excerpts, log lines — it will extract the hashes and ignore labels, commas, whitespace, and prose. Duplicate entries within the same watchlist are silently skipped; the status line reports how many new hashes were added.

### How Matching Works

After every scan completes, all extracted hashes are joined against all watchlist entries in a single SQLite query. Any matching row in the results table is highlighted **red** (white text on a red background). The status bar shows a `⚠ N watchlist hits` count. Watchlist highlights are also applied when loading a historical scan from Scan History.

### Deleting a Watchlist

Select it and click **Delete Selected**. A confirmation prompt will appear. Deletion removes the watchlist and all of its entries and cannot be undone.

---

## Scan History

Every completed scan is automatically saved to `hashharvest.db`. Click **Scan History** to view and reload past scans.

### History Dialog

| Column | Description |
|--------|-------------|
| Date / Time | When the scan ran (truncated to the minute). |
| Directory | Input directory that was scanned. |
| Files | Number of files processed. |
| Hashes Found | Total hashes extracted. |

Use the **Show** drop-down to filter by time range:

| Option | Shows scans from |
|--------|-----------------|
| Today | Midnight of the current day |
| Last 7 days | Rolling 7-day window |
| Last 30 days | Rolling 30-day window (default) |
| Last 90 days | Rolling 90-day window |
| All time | Entire database |

Select a row and click **Load Selected** to restore that scan into the main window. The results table, summary counts, export buttons, and any watchlist highlights are all populated exactly as they would be after a live scan.

---

## Exporting Results

No file is written automatically. After a scan completes (or after loading a historical scan), use the export buttons to save results. Both buttons are disabled until there are results to export.

### Export CSV

Columns: `Absolute_Path`, `Hash_Type`, `Hash_Value`, `Line`, `Context`

```csv
Absolute_Path,Hash_Type,Hash_Value,Line,Context
C:\evidence\report.pdf,MD5,44d88612fea8a8f36de82e1278abb02f,12,[page 1] Hash: 44d88612fea8a8f36de82e1278abb02f found in
C:\evidence\alerts.log,SHA1,da39a3ee5e6b4b0d3255bfef95601890afd80709,47,process exited with hash da39a3ee5e6b4b0d3255bfef9560189
C:\evidence\iocs.json,SHA256,e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855,3,"sha256": "e3b0c44298fc1c149afbf4c8996fb924
```

### Export JSON

A flat array of objects, one entry per hash found.

```json
[
  {
    "absolute_path": "C:\\evidence\\report.pdf",
    "hash_type": "MD5",
    "hash_value": "44d88612fea8a8f36de82e1278abb02f",
    "line": 12,
    "context": "[page 1] Hash: 44d88612fea8a8f36de82e1278abb02f found in"
  },
  {
    "absolute_path": "C:\\evidence\\alerts.log",
    "hash_type": "SHA1",
    "hash_value": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "line": 47,
    "context": "process exited with hash da39a3ee5e6b4b0d3255bfef9560189"
  }
]
```

| Field | Description |
|-------|-------------|
| `Absolute_Path` / `absolute_path` | Full filesystem path of the source file. |
| `Hash_Type` / `hash_type` | Algorithm: `MD5`, `SHA1`, `SHA256`, or `SHA512`. |
| `Hash_Value` / `hash_value` | Lowercase hexadecimal hash string. |
| `Line` / `line` | Line number within the file where the hash first appears. |
| `Context` / `context` | Surrounding text snippet; PDF/PPTX hits include a page or slide prefix. |
