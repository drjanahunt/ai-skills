---
name: rename-invoices
description: >
  Automatically rename and file uncategorised invoice PDFs in a watched expenses folder.
  Use this skill whenever the user says things like "rename my invoices", "organise my
  invoices", "file the new invoices", "sort the PDFs in the invoices folder", or any variation of
  wanting to process, name, or move invoice files into monthly folders. Also trigger when the user
  asks to run this task automatically or on a schedule. This skill handles the full pipeline:
  detecting new unorganised PDFs, reading them, extracting provider and date, renaming to
  YYYY-MM Provider.pdf, and moving to the correct MM-YYYY folder.
---

# Rename & File Invoices

## What this skill does

Scans the root of an invoices folder for PDF files that haven't yet been filed into a monthly
subfolder, reads each one to extract the service provider and invoice date, renames it in the
agreed format, and moves it into the correct monthly folder.

## Folder layout

```
Invoices - expenses 2026/          ← workspace root (mounted folder)
├── 01-2026/
├── 02-2026/
│   └── ...already-filed invoices...
├── ...
├── 12-2026/
└── NewInvoice.pdf                 ← unorganised files sit here, at the root
```

Monthly folders follow the pattern `MM-YYYY`. The final filename format is `YYYY-MM Provider.pdf`
(e.g. `2026-04 ExampleVendor.pdf`). This date-first format means files sort chronologically in Finder.

## Step-by-step workflow

### 1. Detect unorganised files

List only the PDF files sitting directly in the folder root (not inside any subfolder). These are
the candidates for renaming.

```python
import os

folder = "/path/to/Invoices - expenses 2026"
candidates = [
    f for f in os.listdir(folder)
    if f.lower().endswith(".pdf") and os.path.isfile(os.path.join(folder, f))
]
```

If there are no candidates, tell the user there's nothing to do and stop.

### 2. Read each PDF

**Critical:** Files must be fully downloaded locally — not cloud-only Google Drive stubs —
before they can be read. If a file fails with `OSError: [Errno 35] Resource deadlock avoided`
(EDEADLK), it is still a Drive cloud stub. Skip it, collect its name, and tell the user at the
end which files couldn't be processed and why.

Use `pypdf` with a `BytesIO` buffer (direct streaming from a FUSE-mounted drive can deadlock):

```python
import io
from pypdf import PdfReader

def read_pdf_text(path):
    try:
        with open(path, "rb") as fh:
            data = fh.read()          # read fully into memory first
        reader = PdfReader(io.BytesIO(data))
        return "\n".join(page.extract_text() or "" for page in reader.pages)
    except OSError as e:
        if e.errno == 35:
            return None   # still a cloud stub — caller will skip and flag
        raise
```

### 3. Extract provider name and invoice date

Use the full extracted text to identify:

- **Provider name** — look for known recurring providers first (see table below), then fall back
  to reading the document header. Use short, clean names without legal suffixes
  (e.g. `ExampleVendor`, not `ExampleVendor Accounting s.r.o.`).
- **Invoice date** — the date the invoice was issued (not the due date or the billing period start).
  Format as `YYYY-MM`.

**Known providers table (example):**

| Provider | Clues in PDF |
|---|---|
| Example SaaS Co | distinctive company name and domain anywhere in the text |
| Example Cloud Provider | provider name + billing email domain, numeric invoice ID pattern |
| Example Accounting Firm | firm name + "Faktura - daňový doklad" (Czech tax invoice marker) |

This table is meant to be edited — add your own recurring vendors here as you encounter them,
using whatever distinctive text appears reliably on their invoices (company name, billing email
domain, invoice number pattern).

If the provider cannot be determined from the text, flag the file and ask the user.

### 4. Build the new filename

```python
new_name = f"{year}-{month:02d} {provider}.pdf"
# e.g. "2026-04 ExampleVendor.pdf"
```

### 5. Confirm before renaming

Always show the proposed changes and wait for confirmation before renaming or moving any file.
Present a clear table:

```
I found 2 invoices to rename:

  Faktura 269803475.pdf       → 2026-04 ExampleAccountingFirm.pdf  → 04-2026/
  Invoice-AB12CD34-0001.pdf   → 2026-04 ExampleCloudProvider.pdf   → 04-2026/

Shall I go ahead?
```

### 6. Rename and move

After the user confirms:

```python
import shutil

src = os.path.join(folder, old_name)
dst = os.path.join(folder, f"{month:02d}-{year}", new_name)
os.rename(src, dst)
```

If the target month folder doesn't exist yet, create it first:

```python
os.makedirs(os.path.join(folder, f"{month:02d}-{year}"), exist_ok=True)
```

### 7. Report

List every file that was renamed and moved. If any files were skipped (cloud stubs or unreadable),
list those too with a note that the user should make sure they are downloaded locally before
running again.

## Handling Google Drive cloud stubs

Files saved in Google Drive but not yet downloaded locally appear in the folder listing but
return `EDEADLK` when read. The workaround: open the file in Google Drive locally to force it to
download, then run the task again. Files in subfolders are unaffected — only root-level files in
a watched Drive folder exhibit this behaviour.

## Recurring providers reference

See the table in Step 3. When processing a new unknown provider, add it to that table in this
skill for next time (or ask the user to confirm the short name to use).
