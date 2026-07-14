---
name: pdf-deep-translator
description: Deep PDF-to-Markdown extraction and faithful chunked translation workflow for long technical PDFs, standards, papers, manuals, and scanned documents. Use when the user asks to convert a PDF to Markdown with preserved chapter hierarchy, tables, and relative image links, then translate the whole document accurately into Chinese or another language, especially when MinerU or another PDF extraction tool may be needed. Also trigger when the user says pdf_deep_translator.
---

# PDF Deep Translator

## Core Contract

Use this skill to extract a PDF into high-fidelity Markdown, then translate the extracted Markdown in bounded chunks without loading the whole source into context.

Preserve the user's requirements exactly. For standards and technical documents, translate sentence by sentence, with accurate terminology, without adding, deleting, summarizing, or paraphrasing content beyond necessary language conversion.

## Workflow

1. Confirm the source PDF path, target language, output folder, whether to keep the extracted source Markdown, and whether to translate the full document or selected sections.
2. Check PDF extraction dependencies.
3. Extract the PDF to Markdown and image assets.
4. **Domain Detection & Terminology Research** (see dedicated section below).
5. Inspect the user's available translation options and ask them to choose.
6. **Plan translation chunks** based on chapter boundaries (see "Chunk Planning" below).
7. **Launch parallel sub-agent tasks** for each chunk (see "Sub-Agent Translation" below).
8. Wait for all sub-agents to complete, then merge chunks into the final translated file.
9. Validate output structure, table formatting, image paths, and final coverage.

Ask clarifying questions before action when the output location, extraction method, translation method, or scope is ambiguous.

## Dependency Check And Extraction Tool Choice

First check whether MinerU is available locally. Look for installed CLIs, Python packages, Node packages, or an existing MinerU skill/tool configuration. Use commands appropriate to the environment, such as:

```powershell
Get-Command mineru -ErrorAction SilentlyContinue
python -m pip show mineru mineru-open-api 2>$null
```

or in linux/macOS:

```bash
command -v mineru 2>/dev/null
python3 -m pip show mineru mineru-open-api 2>/dev/null
```

If MinerU is not available, ask the user to choose one:

- Register/use MinerU.
- Use another lightweight PDF conversion tool they specify.

If the user chooses MinerU registration, explain briefly:

- MinerU is provided by OpenXLab/MinerU for document parsing and PDF-to-Markdown conversion.
- The user should register/login on the official MinerU/OpenXLab service, create or locate an API token, and provide that token for extraction.
- Treat tokens as sensitive. Do not print the token back unnecessarily. Prefer environment variables or local config if a CLI supports them.

If the user chooses another lightweight PDF conversion tool, follow the user's tool preference and preserve the same Markdown output requirements below. Do not switch tools silently.

## Markdown Extraction Requirements

Create or keep an English/source Markdown file before translation when the user asks for it.

Preserve the PDF's chapter hierarchy in Markdown headings as closely as possible:

- PDF chapter/section levels should map to corresponding Markdown heading levels.
- Do not flatten nested sections.
- Keep figure captions and table captions near their source objects.
- Keep code blocks, grammar productions, identifiers, signal names, and command examples structurally intact.

Tables must be Markdown tables whenever feasible. If the extractor produces HTML tables, convert them to Markdown tables unless doing so would lose complex row/column semantics. For complex tables, keep valid HTML and explain only if needed.

Images must be stored in a same-name asset folder next to the Markdown file and referenced by relative path. Example target style:

```markdown
![Figure 3—TAP state and enable signal timing](IEEE-1687-2014/pic_1.jpg)
```

Do not use absolute image paths in the deliverable unless the user explicitly requests them.

---

## Domain Detection & Terminology Research

**This step runs AFTER extraction and BEFORE translation.** It is mandatory for technical documents.

### Step 1: Detect the domain

Read the first 100 lines of the source Markdown file. Identify the document's technical domain based on:
- Title, keywords, section headings
- Repeated technical terms, acronyms, product names
- Publisher/author (e.g., Siemens EDA → semiconductor/DFT, ARM → CPU/architecture)

Summarize the detected domain in one line, e.g.:
- "Semiconductor DFT / IEEE 1687 IJTAG"
- "Cloud computing / Kubernetes orchestration"
- "Machine learning / transformer architecture"

### Step 2: Search for domain terminology

Use web search to find domain-specific bilingual terminology tables. Run 2-3 searches with queries like:
- `"{domain} 专业术语 中文翻译 对照表"`
- `"{domain} terminology glossary 中英对照"`
- `"{specific_standard} 术语 翻译"`

Also fetch one high-quality reference page (e.g., a CSDN or Zhihu article explaining the domain's key concepts) to extract real-world translation conventions used by Chinese practitioners.

### Step 3: Generate the glossary file

Create a glossary file at `.openclaw/tmp/translation/glossary.md` using the format in `references/glossary-template.md`.

**Terminology classification rules:**

| Category | Action | Examples |
|---|---|---|
| Abbreviations / acronyms | **Keep English**, add Chinese annotation on first occurrence | ICL（仪器连接语言）, CPU（中央处理器）, API |
| ALL-CAPS terms / standard names | **Keep English** | IEEE 1687, Verilog, VHDL, Linux |
| Tool / product names | **Keep English** | Tessent Shell, Synopsys DC, Cadence |
| Code / commands / file paths | **Keep English** | `set_context patterns -ijtag`, `*.v` |
| Technical terms (lowercase/mixed) | **Translate**, keep English in parentheses | scan chain（扫描链）, cache（缓存）, plug-and-play（即插即用）, retargeting（重定向） |
| Domain-specific verbs/nouns | **Translate**, keep English in parentheses | capture（捕获）, shift（移位）, update（更新） |

When in doubt, present the glossary to the user for confirmation before proceeding to translation.

### Step 4: User confirmation

Present the glossary to the user and ask:
- Are there terms that should be added?
- Are there terms that should be reclassified (keep English vs. translate)?
- Any custom preferences?

Only proceed to translation after the user confirms the glossary.

---

## Translation Method Selection

Before translating, inspect available local translation tools. Look for common local CLIs, Python packages, scripts, or services the user may have installed. Examples:

```powershell
Get-Command trans,argos-translate,ollama,deepl,translate-shell -ErrorAction SilentlyContinue
python -m pip list | Select-String -Pattern 'argostranslate|transformers|sentencepiece|deepl|openai'
```

or in linux/macOS:

```bash
command -v trans argos-translate ollama deepl translate-shell 2>/dev/null
python3 -m pip list 2>/dev/null | grep -i 'argostranslate\|transformers\|sentencepiece\|deepl\|openai'
```

Then present the user with the discovered local tools plus one additional option:

- Use the current large language model directly.

If the user selects a local translation tool, call that tool directly and continue preserving all formatting requirements. Do not use network translation services unless the user explicitly chooses them.

If the user selects the current large language model, use the chunked translation method below.

---

## Chunk Planning

Before launching sub-agents, plan the chunk boundaries:

1. Run `grep -n "^## [0-9]" <source_file>` to find chapter-level headings.
2. Split the file into chunks based on chapter boundaries. Each chunk should be **800-1500 lines** (adjust based on total file size).
3. For chapters longer than 1500 lines, split at major sub-headings (`##` level).
4. Record the line ranges for each chunk.

Example chunk plan for an 8500-line file:

| Chunk | Lines | Content |
|---|---|---|
| chunk_01 | 1-472 | Front matter + Chapter 1 |
| chunk_02 | 473-1299 | Chapter 2 |
| chunk_03 | 1300-2200 | Chapter 3 (part 1) |
| ... | ... | ... |

Present the chunk plan to the user for confirmation if they requested it, otherwise proceed directly.

---

## Sub-Agent Translation

Use `sessions_spawn` to create parallel sub-agent tasks for each chunk. Each sub-agent runs independently and writes its output to a temporary file.

### Sub-Agent Task Template

For each chunk, spawn a sub-agent with the following task structure:

```
你是 {domain} 技术文档翻译专家。请执行以下翻译任务：

**翻译规则：**
1. 所有专业术语保留英文原文，不翻译。术语表见下方。
2. 代码块、命令、文件名、Verilog/HDL/Python 等代码保留英文原文
3. 仅翻译描述性文字/散文部分为中文
4. 逐句翻译，不增减内容
5. 保留 Markdown 格式（标题层级、表格、图片链接等）
6. 术语表中"保留英文"的术语不翻译，"需翻译"的术语按表格中的中文翻译

**术语表（节选）：**
{paste relevant rows from glossary.md}

**你的任务：** 翻译源文件第 {start_line}-{end_line} 行（{chapter_description}）

**步骤：**
1. 读取源文件 `{source_path}` 的第 {start_line}-{end_line} 行
2. 按规则翻译
3. 将翻译结果写入 `{output_path}`

注意：写入路径的目录如果不存在请创建。翻译要准确、专业。
```

### Parallel Execution

- Spawn all sub-agents simultaneously (they are independent).
- Use `sessions_yield` to wait for all completions.
- Track expected completions. Do NOT poll; wait for push-based completion events.
- After all sub-agents complete, merge all chunk files:

```bash
cat chunk_01.md chunk_02.md ... > {final_output_path}
```

### Merging And Validation

After merging:

1. Verify line count is reasonable (should be close to source line count).
2. Spot-check the first 50 lines, a middle section, and the last 20 lines.
3. Confirm all chapter headings are present in the translated file.
4. Report completion statistics: source lines, translated lines, coverage.

---

## Progress And Resume State

For every chunk, keep a precise resume marker:

```text
source file: <path>
translated file: <path>
completed source lines: <start>-<end>
next source line: <end + 1>
total source lines: <n>
```

If a translated file already exists, inspect its end and any prior progress notes before appending. Never overwrite existing translation unless the user explicitly asks.

Use append-only writes for chunk translation. If a chunk must be corrected, patch only the affected translated range or append a corrected section according to the user's preference.

## Context Compression Rules

If context compression happens during PDF extraction, follow the default summarization rules.

If context compression happens during translation, preserve these requirements verbatim in the summary:

- 翻译要求逐字逐句翻译，用词准确，专业词汇翻译精准，不自己增减内容。
- 翻译后的 Markdown 文件的层次结构需要和目录/PDF 中的保持一致。
- 表格以 Markdown 格式展现。
- 图片放在同名文件夹中，以相对路径表示。
- 不使用未经用户选择的外部翻译软件或网络翻译服务。
- 翻译时按需读取，不需要将所有等待翻译的内容都加入到上下文中。
- 保留源 Markdown，翻译输出到新的目标 Markdown 文件。
- 继续时必须从明确记录的下一源行开始。
- 术语表见 `./glossary.md`。

Already translated content may be omitted from the compressed context. Keep only:

- Source file path, translated file path, asset folder path.
- Last completed source line and next source line.
- User-selected extraction tool and translation method.
- Professional glossary and term decisions.
- Any formatting quirks that must be maintained.

Use `references/glossary-template.md` as the compact glossary shape when a term list is useful.

## Validation

Before reporting completion:

- Compare source and translated progress markers.
- Confirm all requested source lines/sections were translated.
- Check that image references are relative and point into the same-name asset folder.
- Check that obvious extractor HTML tables were converted or intentionally preserved.
- Check that the translated Markdown opens as valid Markdown.

Final responses should be concise and include the output path, completed coverage, and any unresolved limitations. After completing all the tasks, the temporary files need to be deleted, such as the file named './glossary.md'.
