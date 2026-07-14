# pdf-deep-translator

`pdf-deep-translator` is a reusable agent skill for high-fidelity PDF-to-Markdown conversion and segmented translation of long-form technical documents, standards, papers, manuals, and scanned PDFs.

It guides an agent to:

- Check whether MinerU is available before extraction;

- Ask the user whether to register/use MinerU or use another lightweight PDF conversion tool;

- Preserve the PDF's chapter/section hierarchy in Markdown, render tables as Markdown tables where feasible, and store images in a same-name asset folder;

- Automatically detect the document's domain and research bilingual terminology mappings for that field, generating a bilingual glossary to ensure translation consistency;

- Check available local translation tools before translating;

- Plan translation chunks by section boundaries and launch parallel sub-agent translation tasks without loading the entire document into context;

- Maintain strict translation requirements during context compression.

## Installation

Clone or copy this repository into the skills directory used by your agent platform.

### Windows PowerShell

```powershell
$skills = "<agent-skills-dir>"
New-Item -ItemType Directory -Force -Path $skills | Out-Null
git clone <your-repo-url> "$skills\pdf-deep-translator"
```

If you downloaded the repository as a ZIP file, extract it so the final layout looks like this:

```text
<agent-skills-dir>\pdf-deep-translator\SKILL.md
<agent-skills-dir>\pdf-deep-translator\agents\openai.yaml
<agent-skills-dir>\pdf-deep-translator\references\glossary-template.md
```

### macOS / Linux

```bash
mkdir -p "<agent-skills-dir>"
git clone <your-repo-url> "<agent-skills-dir>/pdf-deep-translator"
```

If the skill does not appear immediately, restart or reload your agent platform after installation.

## Usage

Explicitly ask the agent to use this skill:

```text
Use pdf-deep-translator to extract ./IEEE-1687-2014.pdf into Markdown and translate it into Chinese.
```

You can also describe the task naturally:

```text
Convert this PDF to Markdown, preserving headings, tables, and images, then translate the entire document into Chinese in segments.
```

This skill also includes `pdf_deep_translator` as a trigger alias in the skill description.

# Workflow

```text
┌──────────────────────────────────────────────────────┐
│  1. Confirm PDF path, target language, translation    │
│     scope                                            │
├──────────────────────────────────────────────────────┤
│  2. Check dependencies & extract PDF to Markdown      │
├──────────────────────────────────────────────────────┤
│  3. Domain detection & terminology research           │
│     ├─ Read first 100 lines to identify domain        │
│     ├─ Web-search for bilingual terminology tables    │
│     ├─ Generate glossary.md                           │
│     └─ User confirms glossary                         │
├──────────────────────────────────────────────────────┤
│  4. Plan translation chunks by section boundaries     │
├──────────────────────────────────────────────────────┤
│  5. Parallel sub-agent translation                    │
│     ├─ Launch N sub-agents simultaneously             │
│     ├─ Each independently translates its chunk        │
│     └─ Wait for all to complete                       │
├──────────────────────────────────────────────────────┤
│  6. Merge all chunks → final translated file          │
├──────────────────────────────────────────────────────┤
│  7. Validate structure and coverage                   │
└──────────────────────────────────────────────────────┘
```

## Domain Detection & Terminology Research

This is the core innovation of this skill. Before translation begins:

1. **Domain detection**: The agent reads the first 100 lines to identify the document's technical domain (e.g., semiconductor DFT, cloud computing, machine learning).

2. **Terminology search**: The agent performs 2–3 web searches to find bilingual terminology tables used by practitioners in that domain.

3. **Glossary generation**: A glossary is created at `./glossary.md` with the following classification rules:

| Category | Handling | Example |
|---|---|---|
| Abbreviations / acronyms | **Keep in English**, with Chinese annotation | ICL (Instrument Connectivity Language), CPU (Central Processing Unit), API |
| All-uppercase / standard names | **Keep in English** | IEEE 1687, Verilog, VHDL, Linux |
| Tool / product names | **Keep in English** | Tessent Shell, Synopsys DC, Cadence |
| Code / commands / file paths | **Keep in English** | `set_context patterns -ijtag`, `*.v` |
| Technical terms | **Translate**, with English in parentheses | scan chain（扫描链）, cache（缓存） |
| Domain verbs / nouns | **Translate**, with English in parentheses | capture（捕获）, shift（移位） |

4. **User confirmation**: The glossary is presented to the user for review before translation begins.

## Sub-Agent Parallel Translation

For long documents, the skill uses parallel sub-agent tasks:

- The source file is split into chunks of **800–1500 lines** at section boundaries.
- Each chunk is assigned to an independent sub-agent using a standardized prompt that includes the glossary.
- All sub-agents run simultaneously.
- Once all are complete, the results are merged into the final translated file.

## MinerU Notes

This skill does not include MinerU and does not provide MinerU tokens.

If MinerU is not installed or configured, the agent will ask whether you want to:

- Register/use MinerU and provide a token; or

- Use another lightweight PDF conversion tool of your choice.

Treat MinerU tokens and any translation service credentials as sensitive information. Do not commit them to GitHub.

## Repository Layout

```text
pdf-deep-translator/
├── SKILL.md
├── README.md
├── LICENSE
├── .gitignore
├── agents/
│   └── openai.yaml
└── references/
    └── glossary-template.md
```

## Validation

If you have the skill creation tools installed, validate the skill as follows:

```bash
python path/to/skill-creator/scripts/quick_validate.py path/to/pdf-deep-translator
```

On Windows, if Python reads the Markdown as GBK and fails on Unicode characters, enable UTF-8 for validation:

```powershell
$env:PYTHONUTF8='1'
python path\to\skill-creator\scripts\quick_validate.py path\to\pdf-deep-translator
```

## Disclaimer

- This skill is an agent-oriented workflow guide. It does not include MinerU, Poppler, OCR engines, translation engines, API keys, or model weights.
- Users are responsible for installing and authorizing any extraction or translation tools of their choice.
- Users are responsible for ensuring they have the right to process, extract, and translate the provided PDFs.
- The skill avoids using external translation services unless the user explicitly opts for one.
- Generated translations should be reviewed by a qualified human before use in legal, safety-critical, or standards-compliance contexts.