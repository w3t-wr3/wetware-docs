---
name: wetware-docs
description: |
  Wetware Labs LLC official document template. Use this skill ANY TIME the user asks you to create a document, letter, form, proposal, quote, SOW, report, memo, or any professional deliverable for Wetware Labs. Also trigger when the user says "use the template", "use our header", "make a ___ out of this", "Wetware Labs letterhead", or references the official Wetware paper. This skill contains the exact template file and build process that renders correctly on macOS. ALWAYS use this skill for Wetware Labs documents — never try to recreate the header/footer from scratch.
---

# Wetware Labs Document Template

## What This Is

This skill contains the official Wetware Labs document template — a `.docx` file with the company header (WETWARE logo right-aligned, thin black border, "Labs" text) and footer ("Wetware Labs LLC | Confidential | Page X"). It also contains the official logo PNG.

## Why This Process Matters

Through extensive trial and error, we discovered that the ONLY reliable way to produce documents that render correctly on the user's Mac is to open the original template `.docx` with python-docx, clear the body, and add new content programmatically. Approaches using docx-js, raw XML editing, or building from scratch all failed — the header rendered incorrectly (logo top-left instead of top-right, broken borders, etc.). The python-docx approach preserves the header/footer byte-for-byte because it never touches them.

## Template Details

**Header:**
- WETWARE logo image, right-aligned, with thin black bottom border (sz=2, space=5)
- "Labs" text right-aligned below the border (Arial 16pt, color #555555)

**Footer:**
- "Wetware Labs LLC | Confidential | Page X" centered (Arial 7.5pt, color #555555)
- Thin top border

**Page Layout:**
- US Letter (12240x15840 DXA)
- Margins: top=1800, right=1440, bottom=1200, left=1440
- Header distance: 708, Footer distance: 708

**Typography:**
- Font: Arial throughout
- Section headings: Bold 13pt black with bottom border (sz=3, color=000000, space=3)
- Body text: 10pt, color #333333
- Labels/small text: 9pt, color #555555

## Assets

- `assets/Wetware_Labs_Template.docx` — Clean template with header/footer, empty body
- `assets/wetware_logo.png` — Official WETWARE wordmark (black text, transparent PNG, backwards "E")
- `assets/Wetware_Price_Proposal.docx` — Pre-built Price Proposal template (can be used as a reference or starting point)

## Document Types

This skill supports two document types:

### 1. General Documents (Letters, SOWs, Forms, Memos, etc.)
Use the base template (`Wetware_Labs_Template.docx`) and build content from scratch using the pattern below.

### 2. Price Proposals
Use for client quotes, project proposals, and pricing packages. The Price Proposal follows a specific section structure — see "Price Proposal Structure" below for the full layout and code pattern.

## How to Build a Document

### Step 1: Write the Python script

Use this exact pattern — it's the one that works:

```python
from docx import Document
from docx.shared import Pt, Inches, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml.ns import qn
from copy import deepcopy
import io
from zipfile import ZipFile

TEMPLATE_PATH = "<path-to-skill>/assets/Wetware_Labs_Template.docx"
LOGO_PATH = "<path-to-skill>/assets/wetware_logo.png"

# Open the template
doc = Document(TEMPLATE_PATH)

# Clear body but preserve sectPr (this holds header/footer references)
body = doc.element.body
sect_pr = body.findall(qn('w:sectPr'))
sect_pr_copy = [deepcopy(sp) for sp in sect_pr]
for child in list(body):
    body.remove(child)
for sp in sect_pr_copy:
    body.append(sp)

# === ADD YOUR CONTENT HERE ===
# Use doc.add_paragraph(), doc.add_table(), etc.
# See "Content Helpers" below for styling utilities.

# Save
out_path = "/path/to/output.docx"
doc.save(out_path)

# CRITICAL: Re-inject official logo after save
# python-docx preserves the media files from the template, but we want
# to guarantee the official logo is always used.
buf = io.BytesIO()
with ZipFile(out_path, 'r') as zin, ZipFile(buf, 'w') as zout:
    for item in zin.namelist():
        if item.startswith('word/media/'):
            with open(LOGO_PATH, 'rb') as f:
                zout.writestr(item, f.read())
        else:
            zout.writestr(item, zin.read(item))
with open(out_path, 'wb') as f:
    f.write(buf.getvalue())
```

### Step 2: Content Helpers

Use these helpers for consistent styling across all Wetware Labs documents:

```python
BLACK = RGBColor(0, 0, 0)
DARK_GRAY = RGBColor(0x33, 0x33, 0x33)
GRAY = RGBColor(0x55, 0x55, 0x55)

def add_text(doc, text, size=10, color=DARK_GRAY, bold=False, italic=False,
             align=None, space_before=0, space_after=0):
    p = doc.add_paragraph()
    if align:
        p.alignment = align
    p.paragraph_format.space_before = Pt(space_before)
    p.paragraph_format.space_after = Pt(space_after)
    run = p.add_run(text)
    run.font.name = "Arial"
    run.font.size = Pt(size)
    run.font.color.rgb = color
    run.bold = bold
    run.italic = italic
    return p

def section_heading(doc, text):
    """Section heading with bottom border — matches the SOW style."""
    p = doc.add_paragraph()
    p.paragraph_format.space_before = Pt(18)
    p.paragraph_format.space_after = Pt(8)
    run = p.add_run(text)
    run.font.name = "Arial"
    run.font.size = Pt(13)
    run.font.color.rgb = BLACK
    run.bold = True
    pPr = p._p.get_or_add_pPr()
    pBdr = pPr.makeelement(qn('w:pBdr'), {})
    bottom = pBdr.makeelement(qn('w:bottom'), {
        qn('w:val'): 'single',
        qn('w:sz'): '3',
        qn('w:space'): '3',
        qn('w:color'): '000000'
    })
    pBdr.append(bottom)
    pPr.append(pBdr)
    return p

def add_spacer(doc, space_pt=12):
    p = doc.add_paragraph()
    p.paragraph_format.space_before = Pt(space_pt)
    p.paragraph_format.space_after = Pt(0)
    return p
```

### Step 3: Generate PDF

After creating the .docx, generate a PDF version using reportlab. The PDF needs its own header/footer drawing since it's built separately. Use the same layout constants and draw the logo + "Labs" text + borders to match the docx.

## Price Proposal Structure

When the user asks to create a Price Proposal, quote, or project pricing document, follow this exact section layout. Replace all `[PLACEHOLDER]` values with real client info.

### Section Order

1. **Title** — "PRICE PROPOSAL" (22pt, centered, bold) with thin divider below
2. **Proposal Info** — Proposal number (`PROP-YYYY-XXX`), date (auto-generated), valid through (30 days)
3. **Prepared For / By** — Two-column borderless table. Left: client name, company, email, domain. Right: Kasen Sansonetti, Wetware Labs LLC, management@wetwareofficial.com, wetwareofficial.com
4. **Executive Summary** — 2-3 sentences: what the client needs, what we're proposing, and what's included
5. **Project Overview** — Key/value pairs: Project type, Domain, Target Audience, Style Direction, Target Launch
6. **Scope of Work** (`pageBreakBefore = True`) — Two-column table (Phase | Details) with dark header row (#222222) and alternating row shading (#F5F5F5 / #FFFFFF)
7. **Pricing** — Split into:
   - **WEBSITE PACKAGE** (or primary service) — single-row table with flat-rate price
   - **ADD-ON SERVICES (OPTIONAL)** — three-column table (Service | Description | Price) with `$___` placeholders for unfilled prices
   - **TOTAL** row — shaded #EEEEEE with bold borders
8. **Important Notes** — Bold label + body text for each note (content readiness, branding status, platform details, etc.)
9. **Estimated Timeline** — Three-column table (Timeframe | Phase | Key Deliverables) with italic footnote about timeline variability
10. **Terms & Conditions** — Bold label + body for each term: Payment Schedule (50/50), Revisions (2 rounds, additional at agreed-upon rate), Content, Timeline, Ownership, Hosting & Domain, Confidentiality, Cancellation
11. **Acceptance** — Intro paragraph, checkbox list for add-on selection (`\u2610` character), dual signature block (Client left, Wetware Labs right)

### Price Proposal Code Pattern

Use this pattern for the Pricing section tables:

```python
from docx.enum.table import WD_TABLE_ALIGNMENT

# Website Package table
web_table = doc.add_table(rows=2, cols=2)
web_table.alignment = WD_TABLE_ALIGNMENT.CENTER
for row in web_table.rows:
    row.cells[0].width = Inches(4.5)
    row.cells[1].width = Inches(2.0)

# Dark header row
set_cell_text(web_table.cell(0, 0), "Service", size=9, color=WHITE, bold=True)
set_cell_text(web_table.cell(0, 1), "Price", size=9, color=WHITE, bold=True, align=WD_ALIGN_PARAGRAPH.RIGHT)
for col in range(2):
    set_cell_shading(web_table.cell(0, col), "222222")

# Data row
set_cell_text(web_table.cell(1, 0), "[Service Description]", size=10, color=DARK_GRAY, bold=True)
set_cell_text(web_table.cell(1, 1), "$X,XXX", size=10, color=DARK_GRAY, bold=True, align=WD_ALIGN_PARAGRAPH.RIGHT)
set_cell_shading(web_table.cell(1, 0), "F5F5F5")
set_cell_shading(web_table.cell(1, 1), "F5F5F5")

# Add-On Services table (3 columns)
addon_items = [
    ("[Service Name]", "[Description of what's included]", "$___"),
    # ... more items
]
addon_table = doc.add_table(rows=len(addon_items) + 1, cols=3)
for row in addon_table.rows:
    row.cells[0].width = Inches(2.2)
    row.cells[1].width = Inches(3.0)
    row.cells[2].width = Inches(1.3)

# Total row (separate 1-row table)
total_table = doc.add_table(rows=1, cols=2)
set_cell_text(total_table.cell(0, 0), "TOTAL (Package + Selected Add-Ons)", size=10, color=BLACK, bold=True)
set_cell_text(total_table.cell(0, 1), "$_______", size=10, color=BLACK, bold=True, align=WD_ALIGN_PARAGRAPH.RIGHT)
set_cell_shading(total_table.cell(0, 0), "EEEEEE")
set_cell_shading(total_table.cell(0, 1), "EEEEEE")
```

### Acceptance / Signature Block Pattern

```python
# Checkbox list for add-on selection
addon_names = ["[Add-On 1]", "[Add-On 2]", "[Add-On 3]"]
for name in addon_names:
    p = doc.add_paragraph()
    p.paragraph_format.space_before = Pt(1)
    p.paragraph_format.space_after = Pt(1)
    run = p.add_run(f"\u2610  {name}")
    run.font.name = "Arial"
    run.font.size = Pt(9.5)
    run.font.color.rgb = DARK_GRAY

# Dual signature block (borderless table)
sig_table = doc.add_table(rows=5, cols=2)
# Left: CLIENT — name, signature line, date
# Right: WETWARE LABS LLC — Kasen Sansonetti, CEO, signature line, date
```

### Additional Table Helpers

These helpers are needed for proposals (in addition to the base helpers):

```python
def set_cell_shading(cell, color_hex):
    shading = cell._tc.get_or_add_tcPr()
    shd = shading.makeelement(qn('w:shd'), {
        qn('w:val'): 'clear', qn('w:color'): 'auto', qn('w:fill'): color_hex
    })
    shading.append(shd)

def set_cell_borders(cell, top=None, bottom=None, left=None, right=None):
    tc = cell._tc
    tcPr = tc.get_or_add_tcPr()
    tcBorders = tcPr.makeelement(qn('w:tcBorders'), {})
    for side, val in [('top', top), ('bottom', bottom), ('left', left), ('right', right)]:
        if val:
            border = tcBorders.makeelement(qn(f'w:{side}'), {
                qn('w:val'): 'single', qn('w:sz'): val.get('sz', '4'),
                qn('w:space'): '0', qn('w:color'): val.get('color', '000000')
            })
            tcBorders.append(border)
    tcPr.append(tcBorders)

# Additional colors needed for proposals
WHITE = RGBColor(0xFF, 0xFF, 0xFF)
LIGHT_GRAY = RGBColor(0x99, 0x99, 0x99)
```

## Important Notes

- **NEVER** try to build the header/footer from scratch using docx-js or python-docx — always open the template file
- **ALWAYS** re-inject the logo after python-docx saves (the zipfile step above)
- The official logo file is `wetware_logo.png` in the assets folder — a WETWARE wordmark with backwards "E", black on transparent
- Business email: management@wetwareofficial.com (or kasen@wetwareofficial.com for Kasen specifically)
- Business entity: Wetware Labs LLC
- The user's name is Kasen Sansonetti (CEO)
- For Price Proposals, always use `pageBreakBefore = True` on the Scope of Work heading so page 1 ends with Project Overview
- Add-on service prices should be left as `$___` unless the user provides specific numbers — scope needs to be narrowed down with the client first
