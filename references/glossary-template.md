# Glossary Template For Domain-Specific Translation

Use this format when generating a domain-specific glossary for translation tasks.

## Terminology Classification Rules

| Category | Action | Examples |
|---|---|---|
| Abbreviations / acronyms | **Keep English**, add Chinese annotation | ICL（仪器连接语言）, CPU（中央处理器）, API |
| ALL-CAPS terms / standard names | **Keep English** | IEEE 1687, Verilog, VHDL, Linux |
| Tool / product names | **Keep English** | Tessent Shell, Synopsys DC, Cadence |
| Code / commands / file paths | **Keep English** | `set_context patterns -ijtag`, `*.v` |
| Technical terms (lowercase/mixed) | **Translate**, keep English in parentheses | scan chain（扫描链）, cache（缓存） |
| Domain-specific verbs/nouns | **Translate**, keep English in parentheses | capture（捕获）, shift（移位） |

## Glossary Table Format

| English Term | 中文翻译 | Category | Note |
|---|---|---|---|
| ICL | ICL（仪器连接语言） | abbreviation | Instrument Connectivity Language |
| scan chain | 扫描链 | technical term | Serial chain used for shift access. |
| retargeting | 重定向 | technical term | Translating lower-level instrument procedures to a target module/interface. |
| Tessent Shell | Tessent Shell | tool name | Siemens EDA tool |
| IEEE 1687 | IEEE 1687 | standard name | IJTAG standard |
| capture | 捕获 | domain verb | Scan operation |
| plug-and-play | 即插即用 | technical term | Hot-plug capability |

## Notes

- Only include terms that appear in the source document.
- Remove terms that are not relevant to the specific document being translated.
- Present the glossary to the user for confirmation before starting translation.
- The glossary file should be saved at `./glossary.md`.
- Sub-agent tasks should read this file for terminology reference.
