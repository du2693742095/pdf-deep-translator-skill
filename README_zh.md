# pdf-deep-translator

`pdf-deep-translator` 是一个可复用的 agent 技能，用于高保真地将 PDF 转换为 Markdown，并对长篇技术文档、标准、论文、手册和扫描版 PDF 进行分段翻译。

它指导 agent 执行以下操作：

- 在提取之前检查 MinerU 是否可用；

- 询问用户是否注册/使用 MinerU 或使用其他轻量级 PDF 转换工具；

- 在 Markdown 中保留 PDF 的章节层次结构，在可行的情况下将表格渲染为 Markdown 表格，将图像存储在同名资源文件夹中；

- 自动检测文档所属领域并研究该领域的专业术语对照表，生成双语术语表以保证翻译一致性；

- 在翻译之前检查可用的本地翻译工具；

- 按章节边界规划翻译块，并启动并行子 agent 翻译任务，无需将整个文档加载到上下文中；

- 在上下文压缩期间保持严格的翻译要求。

## 安装

将此存储库克隆或复制到您的 agent 平台所使用的 skills 目录中。

### Windows PowerShell

```powershell

$skills = "<agent-skills-dir>"

New-Item -ItemType Directory -Force -Path $skills | Out-Null

git clone <your-repo-url> "$skills\pdf-deep-translator"

```

如果您下载的仓库是 ZIP 文件，请将其解压缩，最终布局如下：

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

如果技能没有立即显示，请在安装后重启或重新加载你的 agent 平台。

## 使用方法

明确要求 agent 使用此技能：

```text

使用 pdf-deep-translator 将 ./IEEE-1687-2014.pdf 提取为 Markdown 并翻译成中文。

```

您也可以自然地描述任务：

```text

将此 PDF 转换为 Markdown，保留标题、表格和图片，然后分段翻译整个文档成中文。

```

此技能还在技能描述中包含 `pdf_deep_translator` 作为触发别名。

# 工作流程

```text
┌───────────────────────────────────────────┐
│  1. 确认 PDF 路径、目标语言、翻译范围      │
├───────────────────────────────────────────┤
│  2. 检查依赖 & 提取 PDF 为 Markdown        │
├───────────────────────────────────────────┤
│  3. 领域检测与术语研究                     │
│     ├─ 读取前 100 行识别领域               │
│     ├─ 网络搜索中英术语对照表              │
│     ├─ 生成 glossary.md                    │
│     └─ 用户确认术语表                      │
├───────────────────────────────────────────┤
│  4. 按章节边界规划翻译块                   │
├───────────────────────────────────────────┤
│  5. 并行子 Agent 翻译                      │
│     ├─ 同时启动 N 个子 agent               │
│     ├─ 每个独立翻译自己的块                │
│     └─ 等待全部完成                        │
├───────────────────────────────────────────┤
│  6. 合并所有块 → 最终翻译文件              │
├───────────────────────────────────────────┤
│  7. 验证结构与覆盖率                       │
└───────────────────────────────────────────┘
```

## 领域检测与术语研究

这是本技能的核心创新点。翻译开始前：

1. **领域检测**：agent 读取前 100 行，识别文档的技术领域（如半导体 DFT、云计算、机器学习）。

2. **术语搜索**：agent 进行 2-3 次网络搜索，查找中国从业者使用的领域双语术语对照表。

3. **术语表生成**：在 `./glossary.md` 创建术语表，分类规则如下：

| 分类 | 处理方式 | 示例 |
|---|---|---|
| 缩写 / 首字母缩略词 | **保留英文**，附中文注释 | ICL（仪器连接语言）, CPU（中央处理器）, API |
| 全大写 / 标准名称 | **保留英文** | IEEE 1687, Verilog, VHDL, Linux |
| 工具 / 产品名称 | **保留英文** | Tessent Shell, Synopsys DC, Cadence |
| 代码 / 命令 / 文件路径 | **保留英文** | `set_context patterns -ijtag`, `*.v` |
| 技术术语 | **翻译**，括号附英文 | scan chain（扫描链）, cache（缓存） |
| 领域动词/名词 | **翻译**，括号附英文 | capture（捕获）, shift（移位） |

4. **用户确认**：翻译前将术语表展示给用户审阅。

## 子 Agent 并行翻译

对于长文档，技能使用并行子 agent 任务：

- 源文件按章节边界划分为 **800-1500 行**的块。
- 每个块分配给一个独立的子 agent，使用包含术语表的标准化提示词。
- 所有子 agent 同时运行。
- 全部完成后将结果合并为最终翻译文件。

## MinerU 注意事项

此技能不包含 MinerU，也不提供 MinerU 令牌。

如果未安装或配置 MinerU，agent 会询问您是否要：

- 注册/使用 MinerU 并提供 token ；或

- 使用您指定的其他轻量级 PDF 转换工具。

请将 MinerU token 和任何翻译服务凭据视为敏感信息。请勿将其提交到 GitHub。


## 仓库布局

```text

pdf-deep-translator/

├── SKILL.md

├── README.md

├── LICENSE

├── .gitignore

├── agents/

│ └── openai.yaml

└── references/

└── glossary-template.md

```

## 验证

如果您已安装技能创建工具，请按以下方式验证该技能：

```bash
python path/to/skill-creator/scripts/quick_validate.py path/to/pdf-deep-translator
```

在 Windows 上，如果 Python 将 Markdown 作为 GBK 读取并因 Unicode 字符而失败，请使用 UTF-8 启用验证：

```powershell
$env:PYTHONUTF8='1'
python path\to\skill-creator\scripts\quick_validate.py path\to\pdf-deep-translator
```

## 声明

- 此技能是面向 agent 的工作流指南。它不包含 MinerU、Poppler、OCR 引擎、翻译引擎、API 密钥或模型权重。
- 用户负责安装并授权其选择的任何提取或翻译工具。
- 用户负责确保其有权处理、提取和翻译所提供的 PDF。
- 除非用户明确选择外部翻译服务，否则该技能会避免使用外部翻译服务。
- 生成的译文应由合格的人类在法律、安全关键或标准合规用途前进行审阅。
