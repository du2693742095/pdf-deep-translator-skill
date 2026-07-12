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
4. Inspect the user's available translation options and ask them to choose.
5. Translate by chunks, appending to a new translated Markdown file.
6. Track progress by source line/page/chapter so work can resume after interruption.
7. Validate output structure, table formatting, image paths, and final coverage.

Ask clarifying questions before action when the output location, extraction method, translation method, or scope is ambiguous.

## Dependency Check And Extraction Tool Choice

First check whether MinerU is available locally. Look for installed CLIs, Python packages, Node packages, or an existing MinerU skill/tool configuration. Use commands appropriate to the environment, such as:

```powershell
Get-Command mineru -ErrorAction SilentlyContinue
python -m pip show mineru mineru-open-api 2>$null
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
[Figure 3—TAP state and enable signal timing](app://-/IEEE-1687-2014/pic_1.jpg)
```

If the environment or Markdown renderer expects ordinary relative image syntax, use:

```markdown
![Figure 3—TAP state and enable signal timing](IEEE-1687-2014/pic_1.jpg)
```

Do not use absolute image paths in the deliverable unless the user explicitly requests them.

## Translation Method Selection

Before translating, inspect available local translation tools. Look for common local CLIs, Python packages, scripts, or services the user may have installed. Examples:

```powershell
Get-Command trans,argos-translate,ollama,deepl,translate-shell -ErrorAction SilentlyContinue
python -m pip list | Select-String -Pattern 'argostranslate|transformers|sentencepiece|deepl|openai'
```

Then present the user with the discovered local tools plus one additional option:

- Use the current large language model directly.

If the user selects a local translation tool, call that tool directly and continue preserving all formatting requirements. Do not use network translation services unless the user explicitly chooses them.

If the user selects the current large language model, use the chunked translation method below.

## Current-Model Chunked Translation Method

Translate on demand. Do not load all untranslated content into context.

Recommended loop:

1. Determine total source Markdown line count.
2. Choose a chunk size that fits context; if the user specified a size, obey it. For long standards, 120-300 source lines per chunk is practical even if the user says each continuation may cover up to 5000 lines.
3. Read only the next untranslated source range.
4. Translate that range faithfully.
5. Append the translated result to the translated Markdown file.
6. Record progress in the final/update message: translated source lines X-Y; next start line Z.

When translating:

- Translate sentence by sentence.
- Preserve Markdown heading hierarchy.
- Preserve tables as Markdown tables.
- Preserve image links and relative paths.
- Preserve code fences, grammar productions, command syntax, identifiers, signal names, register names, enum symbols, options, and literals unless comments or surrounding prose are clearly natural language.
- Translate comments inside examples only when doing so will not change executable syntax or technical meaning.
- Preserve normative modal verbs precisely: shall = 应, should = 宜/应当 according to context, may = 可以, must = 必须.
- Prefer standard DFT/IJTAG terminology: scan chain = 扫描链, retargeting = 重定向, instrument = 仪器, register = 寄存器, port = 端口, capture/shift/update = 捕获/移位/更新, active scan chain = 活动扫描链.
- Do not add explanations, summaries, warnings, or editorial notes into the translated Markdown unless the source has them.

For standards, keep labels such as `NOTE—`, `Figure N—`, `Table N—`, clause numbers, rule letters, and numbered lists aligned with the source.

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

Final responses should be concise and include the output path, completed coverage, and any unresolved limitations.
