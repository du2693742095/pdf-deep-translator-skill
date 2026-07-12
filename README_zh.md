# pdf-deep-translator

`pdf-deep-translator` 是一个可复用的 agent 技能，用于高保真地将 PDF 转换为 Markdown，并对长篇技术文档、标准、论文、手册和扫描版 PDF 进行忠实的分段翻译。

它指导 agent 执行以下操作：

- 在提取之前检查 MinerU 是否可用；

- 询问用户是否注册/使用 MinerU 或使用其他轻量级 PDF 转换工具；

- 在 Markdown 中保留 PDF 的章节层次结构；

- 在可行的情况下将表格渲染为 Markdown 表格；

- 将图像存储在同名资源文件夹中，并使用相对路径引用它们；

- 在翻译之前检查可用的本地翻译工具；

- 按需分段翻译，无需将整个文档加载到上下文中；

- 在上下文压缩期间保持严格的翻译要求。

## 安装

将此存储库克隆或复制到您的 agent 平台所使用的技能目录中。

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

## 预期工作流程

1. agent 确认 PDF 路径、输出文件夹、目标语言和翻译范围。

2. agent 检查是否存在 MinerU。

3. 如果不存在 MinerU，agent 会询问是否注册/使用 MinerU 或使用其他轻量级转换器。

4. agent 提取 Markdown 和图像资源。

5. agent 会检查本地翻译工具，并询问要使用哪种翻译方法。

6. agent 会分段翻译，并将翻译后的内容追加到新的翻译后的 Markdown 文件中。

7. agent 会按源行范围报告翻译进度，以便安全地恢复翻译。

## MinerU 注意事项

此技能不包含 MinerU，也不提供 MinerU 令牌。

如果未安装或配置 MinerU，agent 会询问您是否要：

- 注册/使用 MinerU 并提供令牌；或

- 使用您指定的其他轻量级 PDF 转换工具。

请将 MinerU 令牌和任何翻译服务凭据视为敏感信息。请勿将其提交到 GitHub。

## Markdown 输出规则

提取的 Markdown 应保留 PDF 的逻辑结构：

- 标题应与 PDF 的章节/节层级结构相匹配；

- 表格应尽可能使用 Markdown 表格；

- 图片应靠近其标题；

- 图片应放置在同名文件夹中；

- 图片链接应为相对路径。

图片引用示例：

```markdown

[图 3—TAP 状态和使能信号时序](app://-/IEEE-1687-2014/pic_1.jpg)

```

如果 Markdown 渲染器期望使用标准图片语法：

```markdown

![图 3—TAP 状态和使能信号时序](IEEE-1687-2014/pic_1.jpg)

```

## 翻译规则

使用当前大型语言模型进行翻译时，该技能指示 agent：

- 逐句翻译；

- 保持技术术语一致；

- 避免添加、删除、概括或编辑；

- 保留 Markdown 标题、表格、链接、代码块、语法、标识符、信号名称、选项和字面值；

- 将翻译后的内容追加到新的目标 Markdown 文件中；

- 仅读取下一个所需的源文本块，而不是加载整个文档。

对于中文技术标准，该技能使用如下术语：

| 源术语 | 首选中文 |

|---|---|

| 扫描链 | 扫描链 |

| 重定向 | 重定向 |

| 仪器 | 仪器 |

| 寄存器 | 寄存器 |

| 端口 | 端口 |

| 捕获-移位-更新 (CSU) | 捕获-移位-更新（CSU） |

| 活动扫描链 | 活动扫描链 |

使用 `references/glossary-template.md` 来维护长篇翻译的精简术语表。

## 上下文压缩行为

在提取过程中，可以接受正常的上下文压缩。

在翻译过程中，该技能要求 agent 保留完整的翻译需求、当前进度标记、所选工具和术语表决策。已翻译的文本可以从压缩的上下文中省略。

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

## 许可

MIT 许可。见 [LICENSE](LICENSE)。