# pdf-deep-translator

`pdf-deep-translator` is a reusable agent skill for high-fidelity PDF-to-Markdown extraction and faithful chunked translation of long technical documents, standards, papers, manuals, and scanned PDFs.

It guides agents to:

- check whether MinerU is available before extraction;
- ask the user whether to register/use MinerU or use another lightweight PDF conversion tool;
- preserve PDF chapter hierarchy in Markdown;
- render tables as Markdown tables where feasible;
- store images in a same-name asset folder and reference them with relative paths;
- inspect available local translation tools before translating;
- translate on demand in chunks, without loading the whole document into context;
- preserve strict translation requirements across context compression.

## Installation

Clone or copy this repository into the skills directory used by your agent platform.

### Windows PowerShell

```powershell
$skills = "<agent-skills-dir>"
New-Item -ItemType Directory -Force -Path $skills | Out-Null
git clone <your-repo-url> "$skills\pdf-deep-translator"
```

If you downloaded the repository as a ZIP file, extract it so the final layout is:

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

Restart or reload your agent platform after installation if the skill does not appear immediately.

## Usage

Ask your agent to use the skill explicitly:

```text
Use pdf-deep-translator to extract ./IEEE-1687-2014.pdf to Markdown and translate it into Chinese.
```

You can also describe the task naturally:

```text
Convert this PDF to Markdown with preserved headings, tables, and figures, then translate the full document into Chinese chunk by chunk.
```

The skill also includes `pdf_deep_translator` as a trigger alias in the skill description.

## Expected Workflow

1. The agent confirms the PDF path, output folder, target language, and translation scope.
2. The agent checks for MinerU.
3. If MinerU is missing, the agent asks whether to register/use MinerU or use another lightweight converter.
4. The agent extracts Markdown and image assets.
5. The agent checks local translation tools and asks which translation method to use.
6. The agent translates in chunks and appends to a new translated Markdown file.
7. The agent reports progress by source line range so translation can resume safely.

## MinerU Notes

This skill does not bundle MinerU and does not provide a MinerU token.

If MinerU is not installed or configured, the agent should ask whether you want to:

- register/use MinerU and provide a token; or
- use another lightweight PDF conversion tool you specify.

Treat MinerU tokens and any translation service credentials as sensitive. Do not commit them to GitHub.

## Markdown Output Rules

The extracted Markdown should preserve the PDF's logical structure:

- headings should match the PDF chapter/section hierarchy;
- tables should be Markdown tables where feasible;
- figures should stay near their captions;
- images should be placed in a same-name folder;
- image links should be relative.

Example image reference:

```markdown
[Figure 3—TAP state and enable signal timing](app://-/IEEE-1687-2014/pic_1.jpg)
```

If the Markdown renderer expects normal image syntax:

```markdown
![Figure 3—TAP state and enable signal timing](IEEE-1687-2014/pic_1.jpg)
```

## Translation Rules

When using the current large language model for translation, the skill instructs the agent to:

- translate sentence by sentence;
- keep technical terms consistent;
- avoid adding, deleting, summarizing, or editorializing;
- preserve Markdown headings, tables, links, code fences, grammar, identifiers, signal names, options, and literals;
- append translated chunks to a new target Markdown file;
- read only the next needed source chunk instead of loading the full document.

For Chinese technical standards, the skill uses terminology such as:

| Source term | Preferred Chinese |
|---|---|
| scan chain | 扫描链 |
| retargeting | 重定向 |
| instrument | 仪器 |
| register | 寄存器 |
| port | 端口 |
| capture-shift-update (CSU) | 捕获-移位-更新（CSU） |
| active scan chain | 活动扫描链 |

Use `references/glossary-template.md` to maintain a compact glossary for long translations.

## Context Compression Behavior

During extraction, normal context compression is acceptable.

During translation, the skill requires the agent to preserve the full translation requirements, current progress markers, selected tools, and glossary decisions. Already translated prose can be omitted from compressed context.

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

If you have the skill creator utilities installed, validate the skill with:

```bash
python path/to/skill-creator/scripts/quick_validate.py path/to/pdf-deep-translator
```

On Windows, if Python reads Markdown as GBK and fails on Unicode characters, run validation with UTF-8 enabled:

```powershell
$env:PYTHONUTF8='1'
python path\to\skill-creator\scripts\quick_validate.py path\to\pdf-deep-translator
```

## Declarations

- This skill is a workflow guide for agents. It does not include MinerU, Poppler, OCR engines, translation engines, API keys, or model weights.
- Users are responsible for installing and licensing any extraction or translation tools they choose.
- Users are responsible for ensuring they have the right to process, extract, and translate the PDFs they provide.
- The skill is designed to avoid external translation services unless the user explicitly chooses one.
- Generated translations should be reviewed by a qualified human for legal, safety-critical, or standards compliance use.

## License

MIT License. See [LICENSE](LICENSE).
