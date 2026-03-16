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
  - - "Labs" text right-aligned below the border (Arial 16pt, color #555555)
   
    - **Footer:**
    - - "Wetware Labs LLC | Confidential | Page X" centered (Arial 7.5pt, color #555555)
      - - Thin top border
       
        - **Page Layout:**
        - - US Letter (12240x15840 DXA)
          - - Margins: top=1800, right=1440, bottom=1200, left=1440
            - - Header distance: 708, Footer distance: 708
             
              - **Typography:**
              - - Font: Arial throughout
                - - Section headings: Bold 13pt black with bottom border (sz=3, color=000000, space=3)
                  - - Body text: 10pt, color #333333
                    - - Labels/small text: 9pt, color #555555
                     
                      - ## Assets
                     
                      - - `assets/Wetware_Labs_Template.docx` — Clean template with header/footer, empty body
                        - - `assets/wetware_logo.png` — Official WETWARE wordmark (black text, transparent PNG, backwards "E")
                         
                          - ## How to Build a Document
                         
                          - ### Step 1: Write the Python script
                         
                          - Use this exact pattern — it's the one that works:
                         
                          - ```python
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

                            ## Important Notes

                            - **NEVER** try to build the header/footer from scratch using docx-js or python-docx — always open the template file
                            - - **ALWAYS** re-inject the logo after python-docx saves (the zipfile step above)
                              - - The official logo file is `wetware_logo.png` in the assets folder — a WETWARE wordmark with backwards "E", black on transparent
                                - - Business email: management@wetwareofficial.com (or kasen@wetwareofficial.com for Kasen specifically)
                                  - - Business entity: Wetware Labs LLC
                                    - - The user's name is Kasen Sansonetti (CEO)
