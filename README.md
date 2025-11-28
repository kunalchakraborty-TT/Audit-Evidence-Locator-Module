Audit-Evidence-Locator automatically finds text inside PDFs, images, and scanned documents, then highlights each match with a red box. It works with OCR, supports Word/Excel files, and keeps the original file unchanged. A simple tool to speed up audits, reviews, and document checks.

Purpose
This repository contains a Python solution for AuditRAM: a program that searches for a text phrase in a file (PDF, image, DOCX, XLSX) and produces an annotated output with red, unfilled bounding boxes around each found occurrence, without altering the original file. The Colab notebook, helper scripts, and example files are included. See the assignment description in the uploaded brief. 

Quick summary (what this does)

Accepts PDF (text or scanned), image (PNG/JPG/TIFF), Word (.docx), and Excel (.xlsx) inputs.

For selectable PDF text: uses PyMuPDF (fitz) to find exact text bounding boxes and add rectangle annotations (no fill).

For scanned PDFs / images: uses Tesseract OCR via pytesseract to get word-level boxes and draws red outline rectangles using Pillow.

For .docx/.xlsx: optionally converts to PDF using LibreOffice headless (soffice) and processes the resulting PDF.

Outputs are new files (original remains unchanged). Example output name pattern: inputname_annotated.pdf or inputname_annotated.png.

Files included (what to expect in repo)

auditram_annotate.py — main script (CLI) to run locally (PyMuPDF + Pillow + pytesseract).

colab_notebook.ipynb (or a sequence of Colab cells in README) — Google Colab version which installs dependencies, accepts uploads, runs annotation, and downloads result.

sample_report.pdf — example report generated for testing.

examples/ — sample images/PDFs used for testing (if present).

This README — instructions and troubleshooting.

Note: If you received the Colab notebook from me (or saw images) the uploaded assignment brief is also included.
Requirements
Python packages (pip)
pip install pymupdf pillow pytesseract python-docx openpyxl

System packages

Tesseract OCR (required for OCR on images/scanned PDFs)

Ubuntu: sudo apt-get install tesseract-ocr

macOS (Homebrew): brew install tesseract

Windows: download installer from tesseract-ocr repo and add to PATH

LibreOffice (optional) — only if you want automatic .docx/.xlsx → .pdf conversion inside the script

Ubuntu: sudo apt-get install libreoffice

Windows/macOS: install LibreOffice and ensure soffice is on PATH

How to run
Option A — Run in Google Colab (recommended if you can't install system deps)

Open Google Colab.

Create a new notebook and paste the Colab cells (installation → helper functions → upload → run).

Run the first cell to install pip packages and tesseract-ocr via apt.

Upload your file using the upload cell and set the search phrase.

Run annotation cell and download the annotated output.

(Colab steps are included in the notebook and are one-click; using Colab avoids local dependency issues.)

Option B — Run locally (Linux / macOS / Windows)

Install Python 3.8+ and pip packages:

pip install pymupdf pillow pytesseract python-docx openpyxl


Install Tesseract OCR (system package). Confirm tesseract runs from command line.

(Optional) Install LibreOffice and ensure soffice is in PATH if you want automatic conversion of .docx/.xlsx.

Use the script (example):

python auditram_annotate.py --input sample.pdf --text "Total Amount" --output sample_annotated.pdf


For images:

python auditram_annotate.py -i invoice_scan.png -t amount -o invoice_scan_annotated.png

Colab quick command snippets (copy into cells)

Install cell

!pip install --upgrade pip
!pip install pymupdf pillow pytesseract python-docx openpyxl
!apt-get update -qq
!apt-get install -y -qq tesseract-ocr
# Optional:
# !apt-get install -y -qq libreoffice


Upload & run (example cell)

from google.colab import files
uploaded = files.upload()
input_file = next(iter(uploaded.keys()))
SEARCH_TEXT = "Total Amount"
# then call provided helper function from the notebook to annotate and download

How the code works (high-level)

PDF path (.pdf):

Open with PyMuPDF.

For pages with selectable text: page.search_for(search_text) to get bounding boxes; add rectangle annotations (page.add_rect_annot) with red stroke and no fill.

For pages that appear image-only (no selectable text): render to an image, run OCR (pytesseract), draw rectangles on the image and convert back to PDF.

Image path (.png, .jpg, ...):

Load with Pillow.

pytesseract.image_to_data() returns word-level bounding boxes (left, top, width, height).

Sliding-window approach to compose multi-word phrases and compute a combined bounding box for the matched words.

Draw outline rectangle (ImageDraw.rectangle) with outline="red" and save annotated image.

.docx / .xlsx:

Convert to PDF using LibreOffice soffice --headless --convert-to pdf into a temp folder, then process that PDF as above.

Example usage(s)

Annotate a PDF (case-insensitive by default):

python auditram_annotate.py --input report.pdf --text "Invoice No" --output report_annotated.pdf


Annotate a PNG:

python auditram_annotate.py -i invoice.png -t "amount" -o invoice_annotated.png


Convert & annotate DOCX (requires LibreOffice):

python auditram_annotate.py -i bill.docx -t "Total Amount"
# produces bill_annotated.pdf next to original

Important implementation details & tips

Case-sensitivity: Default search is case-insensitive. Use --case-sensitive flag in CLI to require exact casing.

Multi-word phrases: The image OCR uses sliding-window joining of adjacent OCR words (configurable max length) to find multi-word phrases.

Scanned PDFs: Scanned pages are rasterized at a zoom factor (default 2.0) to improve OCR detection. Increase zoom for better accuracy at the cost of memory/time.

File paths on Windows: If you hardcode Windows paths in Python, avoid \ escape issues. Use raw strings r"C:\path\file.pdf" or forward slashes "C:/path/file.pdf" or pathlib.Path.

ModuleNotFoundError: No module named 'fitz' — install PyMuPDF (not the package named fitz):

pip install pymupdf


Tesseract not found — install system tesseract-ocr and ensure it is in your PATH. On Windows, set pytesseract.pytesseract.tesseract_cmd to the full executable path if needed.

LibreOffice not found — convert office files to PDF offline or install soffice on PATH before using .docx/.xlsx conversion.

Troubleshooting

If the script raises ModuleNotFoundError: No module named 'fitz', install pymupdf:

python -m pip install pymupdf


If OCR misses matches: try preprocessing (increase contrast, convert to grayscale, denoise) or increase rendering zoom for scanned PDFs.

If you see long install times on Colab: LibreOffice package is large — install only if necessary.

For very large PDFs: process page-by-page in a loop to avoid high memory usage.

Testing & verification

Use the included example sample_report.pdf or the generated invoice PNGs to test detection of "Amount" and "Total Amount".

Verify output:

PDFs: open with a normal PDF viewer — red rectangles should show as annotations (they are overlay annotations).

Images: open the annotated image to confirm red outlined boxes around matched text.

Suggested improvements (future work)

Add fuzzy matching for OCR errors (e.g., difflib or Levenshtein distance).

Preserve original page ordering when mixing text and scanned pages (current strategy may combine text pages then scanned pages).

Add GUI or web front-end for drag-and-drop processing.

Add multi-language OCR support (pass lang to pytesseract if tesseract language data installed)
