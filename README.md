# Wetware Labs Document Template

Official document template skill for **Wetware Labs LLC**. Generates professional documents with the Wetware Labs header, footer, and branding — works with any AI agent.

## Install

```bash
npx skills add w3t-wr3/wetware-docs
```

Works with: Claude Code, Cursor, Windsurf, Gemini CLI, GitHub Copilot, and any agent supporting the [skills](https://github.com/vercel-labs/skills) standard.

## What It Does

Once installed, just ask your AI to create any document — proposal, SOW, invoice, demand letter, intake form, report — and it will automatically use the official Wetware Labs template:

- **Header**: WETWARE logo (right-aligned) with "Labs" text and thin black border
- - **Footer**: "Wetware Labs LLC | Confidential | Page X" centered
  - - **Typography**: Arial throughout, consistent heading/body styles
    - - **Page Layout**: US Letter, professional margins
     
      - ## Usage
     
      - After installing, just tell your AI what you need:
     
      - - "Make me a proposal for [client name]"
        - - "Create an invoice for [project]"
          - - "Draft a demand letter to [recipient]"
            - - "Build a website intake form"
             
              - The skill handles the template, formatting, and logo automatically.
             
              - ## What's Inside
             
              - ```
                skills/wetware-docs/
                  SKILL.md              # Instructions for the AI agent
                  assets/
                    Wetware_Labs_Template.docx   # Blank template with header/footer
                    wetware_logo.png             # Official WETWARE wordmark
                ```

                ## Manual Use

                Don't use AI? Just download `Wetware_Labs_Template.docx` from the `assets/` folder, open it in Word or Google Docs, and start typing. The header and footer are already set up.

                ## Built by

                **Wetware Labs LLC** — Branding, Software Development, Web Design, SEO & Automation

                management@wetwareofficial.com
