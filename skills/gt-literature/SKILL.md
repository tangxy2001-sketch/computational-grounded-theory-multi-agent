---
name: gt-literature
description: 扎根理论文献对话技能。文献收集复用工具链（paper-search-mcp、brave-search、book-downloader → markitdown → mcp-obsidian），支持数据与文献双向迭代对话，包含隐喻分析方法论、文献搜索策略、文献对话备忘录模板。触发词：文献、论文、理论框架、学术搜索、隐喻分析、概念隐喻、理论敏感性、sensitizing concepts、文献综述、理论对话。
---

# 扎根理论文献对话

本技能管理扎根理论研究中数据与文献的双向迭代对话。全局的研究背景参见 grounded-theory 技能（在 available_skills 列表中查找）。

本技能包含两个阶段：
1. **文献收集**：搜索 → 下载 PDF → 转换 → 存储到知识库
2. **GT 对话**：研究者与 Coding Agent 共读 → 撰写 references 和 MEMO-LIT

项目特定的研究问题和理论对话方向定义在项目目录的 `CLAUDE.md` 中。本技能启动时应读取该文件获取项目上下文。

在建构主义扎根理论中，文献不是用来"验证"或"套用"的，而是用来"对话"的。文献对话是双向的——数据指导你去读什么文献，文献反过来指导你如何更深入地分析数据。

---

## 阶段一：文献收集工作流

根据文献类型选择对应工具和模式。以下工具表为推荐配置，请根据你实际安装的 MCP servers 进行调整。

### 工具参考表

| 工具 | 用途 | 何时使用 |
|------|------|----------|
| `paper-search-mcp` | 搜索英文学术论文 | arXiv/PubMed/bioRxiv/Google Scholar |
| `brave-search` | 搜索中文学术文献 + 综述 + 网页 | CNKI/知网、中文综述、灰色文献 |
| `book-downloader` | 搜索和下载理论专著 | 需要深入某个理论传统时 |
| `firecrawl` | 抓取网页内容转 Markdown | 需要保存网页文献时；PDF下载失败时降级抓取论文网页全文 |
| `markitdown` | PDF/DOCX 转 Markdown | 下载 PDF 后转换为可读文本 |
| `mcp-obsidian` | **只读** Obsidian vault | 搜索/读取已有笔记（如已安装） |
| Write 工具 | 写入知识库 | 直接写 `.md` 文件到 vault 目录 |

**降级策略：** 如果 MCP 工具不可用，降级为 WebSearch 或 WebFetch。

**知识库写入方式：** 如果使用 Obsidian 作为知识库，需配置 vault 路径。在项目 CLAUDE.md 或你的 `~/.claude/CLAUDE.md` 中设置 vault 路径（如 `OBSIDIAN_VAULT_PATH=/path/to/your/vault`）。笔记创建使用 Claude Code 内置 Write 工具直接写入 vault 目录。PDF 下载使用 `paper-search` 的 `download_paper`，`savePath` 设为 vault 的 `Resources/Papers/` 子目录。

> **配置提示：** 你需要将以下路径替换为你自己的 Obsidian vault 路径：
> - 论文笔记目录：`<YOUR_OBSIDIAN_VAULT_PATH>/Papers/`
> - PDF 存储目录：`<YOUR_OBSIDIAN_VAULT_PATH>/Resources/Papers/`
> - 书籍笔记目录：`<YOUR_OBSIDIAN_VAULT_PATH>/Books/`
> - 网页剪藏目录：`<YOUR_OBSIDIAN_VAULT_PATH>/Clippings/`

### 模式1：论文搜索与收集（默认）

```
paper-search-mcp → 下载 PDF → markitdown → Write 工具写入 vault → BibTeX
```

**步骤：**

1. **搜索**：使用 `paper-search-mcp` 搜索 arXiv / PubMed / bioRxiv / Google Scholar
   - 展示 top 5 结果，包含标题、作者、年份、摘要
   - 询问用户选择哪些论文进行处理（或按指示全部处理）

2. **下载**：下载选定论文的 PDF
   - 使用 `paper-search` 的 `download_paper`，`savePath` 设为你的 vault `Resources/Papers/` 目录
   - 文件命名：`[Author]_[Year]_[ShortTitle].pdf`
   - **PDF下载失败时的降级**：使用 `firecrawl` 抓取论文网页全文，保存为 `[Author]_[Year]_[ShortTitle]_web.md`

3. **转换**：使用 `markitdown` 将 PDF 转换为 Markdown 文本；如果步骤2已降级为网页剪裁，则跳过此步

4. **创建笔记**：基于转换结果，提取 Summary、Key Findings、Methodology 等实质内容，按下方 Paper Note Template 填写后，使用 Write 工具写入 vault `Papers/` 目录
   - **禁止创建空壳笔记**：每个字段必须基于论文实际内容填写，不允许留"待补充"占位符

5. **引用**：生成 BibTeX 条目并包含在笔记中

### 模式2：理论专著收集（`book:` 前缀）

```
book-downloader → markitdown → Write 工具写入 vault
```

### 模式3：网页文献收集（`web:` 前缀）

```
brave-search / WebSearch → firecrawl / WebFetch → Write 工具写入 vault
```

### GT 搜索策略（覆盖默认搜索行为）

在本技能中，搜索关键词的选择遵循 GT 原则：
- 用涌现的**概念**作为关键词，而非用研究问题直接搜索（从数据出发）
- 同时搜索中文和英文文献，注意中英文学术话语可能使用不同的概念名称
- 关注综述文章（review article），快速了解领域全景
- 关注该概念的"经典文献"和"最新进展"两端

### Obsidian 笔记模板

#### Paper Note Template

```markdown
---
title: "[Full Paper Title]"
authors: [Author1, Author2, ...]
year: YYYY
source: "[arXiv/PubMed/etc]"
url: "[paper URL]"
pdf: "[[Resources/Papers/filename.pdf]]"
tags: [paper, topic1, topic2]
date_added: YYYY-MM-DD
---

## Summary

[2-3 sentence summary of the paper's main contribution]

## Key Findings

- [Finding 1]
- [Finding 2]
- [Finding 3]

## Methodology

[Brief description of the approach/method]

## Notes

[Space for user's personal notes]

## Questions

- [Open questions or things to follow up on]

## Citation

\`\`\`bibtex
@article{author2024shorttitle,
  title={Full Title},
  author={Author1 and Author2},
  journal={Journal/Conference},
  year={2024},
  url={URL}
}
\`\`\`
```

#### Book Note Template

```markdown
---
title: "[Book Title]"
authors: [Author]
year: YYYY
isbn: "[if available]"
tags: [book, topic1, topic2]
date_added: YYYY-MM-DD
status: unread
---

## About

[Brief description of the book and why it's relevant]

## Key Takeaways

- [Takeaway 1]
- [Takeaway 2]

## Chapter Notes

### Chapter 1: [Title]

[Notes]

## Quotes

> [Notable quotes]

## Citation

\`\`\`bibtex
@book{author2024title,
  title={Book Title},
  author={Author},
  year={2024},
  publisher={Publisher}
}
\`\`\`
```

### 重要说明

- 搜索结果展示后先等用户选择，再下载——让用户决定处理哪些文献
- 如果论文/书籍已在 vault 中存在，通知用户而非重复创建
- 生成 BibTeX 时确保所有字段基于实际元数据，不要编造
- 中文标题保留原标题，有英文翻译时附加
- **笔记必须包含实质内容**：通过 markitdown 转换 PDF 后提取摘要、发现、方法等，不允许创建空壳占位笔记

---

## 项目本地存储（GT 审计追踪）

除知识库（长期跨项目知识库）外，GT 审计追踪文件存储在项目本地：

| 文件类型 | 路径 | 用途 |
|---------|------|------|
| 文献建议清单 | `literature/references_*.md` | 按触发概念组织的文献清单 |
| 文献对话备忘录 | `memos/literature/MEMO-LIT-*.md` | GT 审计追踪，记录文献与数据的对话 |
| 过程笔记 | `memos/literature/lit_dialogue_notes.md` | 文献对话中的临时发现 |

文献建议清单格式：

```markdown
# 文献阅读建议 [日期]

## 触发概念：[从数据中涌现的概念名称]

| 优先级 | 作者(年份) | 标题 | 相关性说明 | 可能为数据分析提供的新问题 |
|--------|------------|------|------------|--------------------------|
| ★★★ | Author (Year) | Title | 为什么相关 | 读完后可以追问什么 |
| ★★ | Author (Year) | Title | 为什么相关 | 读完后可以追问什么 |
```

"可能为数据分析提供的新问题"这一列很重要——它提前预设了文献反向指导数据的路径。

---

## 阶段二：GT 文献对话

文献收集完成后，进入扎根理论的文献对话阶段。

### 方向一：数据 → 文献（数据指导文献阅读）

**何时触发：**
- 初始编码中涌现出重要概念，需要看前人是否讨论过类似现象
- 概念之间的关系不清晰，需要理论框架帮助思考
- 需要"理论敏感性"来深化分析
- 研究者自己提到"这让我想到某个理论"

**操作方式：**
1. 根据涌现的概念，执行阶段一的文献收集工作流
2. 优先搜索中英文学术文献，关注高引用文献
3. 生成文献阅读建议清单（保存到项目本地 `literature/`）
4. 研究者与 Coding Agent 共读文献
5. 讨论文献中的理论框架如何帮助或挑战当前的分析
6. 撰写文献对话备忘录（保存到 `memos/literature/`）

### 方向二：文献 → 数据（文献指导数据分析和理论方向）

这是常被忽视但极其重要的方向。读完文献后，文献可以在三个层面反向指导分析。

#### 层面1：概念敏感化（sensitizing concepts）

文献中的概念可以成为你重新审视数据的"透镜"。

**什么是概念敏感化？** 简单说，就是文献给了你一个新的"提问角度"。例如你读到某个理论概念，回到数据中就可以追问：参与者描述的某种体验是否正因为研究对象提供了某种特定的关系质量？

**关键区分：** 这不是把文献"套"在数据上，而是用文献的概念作为**提问的工具**。你带着新问题回到数据，但数据可能回答"是"，也可能回答"不是，实际上更复杂"。

#### 层面2：理论方向的聚焦

当多个可能的理论方向并存时，文献帮助判断哪个方向理论贡献更大。

**具体做法：** 搜索涌现概念相关的综述文章（review article），看看已有研究做到了哪里，你的发现在哪里可以补充或挑战。理论贡献最大的方向通常是：已有文献讨论过类似现象但你的数据揭示了不同的机制或条件，或者已有文献在其他语境中建立的理论在你的数据语境中需要修正。

#### 层面3：关系框架的启发

文献中的理论模型可以启发你思考概念间关系。

例如某个社会学理论的核心框架，可以启发你追问：数据中的现象是否呈现了该框架预期的模式？又或者数据揭示了该框架未能解释的维度？具体的理论对话方向参见项目 `CLAUDE.md`。

#### 文献反向指导的操作流程

**方式A：研究者读完文献后与Claude讨论**
1. 研究者告诉Claude读了什么文献、有什么感想
2. Claude帮助提炼出2-3个可用于"重新提问"的概念或框架
3. 共同讨论这些新问题如何带回数据

**方式B：研究者将文献提供给Claude**
1. 研究者上传PDF或粘贴关键段落
2. Claude阅读后提炼核心观点
3. Claude输出一份"文献对话备忘录"（见下方模板）

**方式C：Claude主动搜索文献**
1. 编码中涌现重要概念时，Claude执行阶段一的文献收集工作流
2. Claude阅读摘要后生成文献建议清单
3. 将文献笔记写入知识库（含实质内容），文献建议清单保存到项目本地
4. 研究者选择性阅读，然后回到方式A或B

三种方式交替使用。A和B由研究者驱动，C由Claude驱动。

#### 文献反向指导后的具体行动

1. 带着新问题回到已编码的数据中做**二次编码**（re-coding）——不是推翻之前的编码，而是看能否发现之前没注意到的维度
2. 用新的"透镜"指导下一轮**理论抽样**：从BERTopic结果中选取能回答新问题的数据
3. 撰写**文献对话备忘录**（见下方模板）

#### 关键警戒

文献用于"对话"而非"验证"或"套用"。当你从文献中获得新视角回到数据时，同时问两个问题：
- 数据是否支持这个看法？
- 数据在哪里挑战或超越了这个看法？

如果你发现自己只在找"支持"文献观点的数据而忽略矛盾数据，Claude应以方法论顾问角色温和提醒。

---

## 隐喻分析的系统方法

当研究问题涉及参与者如何理解和建构某个对象时，**隐喻**是核心分析材料。隐喻不只是修辞手法，而是认知方式（Lakoff & Johnson, 1980）——人们通过已知概念来理解未知事物。

### 识别隐喻

在初始编码中，凡是参与者用以下结构描述研究对象的地方，标注为隐喻型编码：
- "当作X""像X一样""就是我的X"（显性隐喻）
- 将研究对象的特征描述为另一类事物的典型特征（隐性隐喻）
- 对研究对象的情感反应与对另一类事物的情感反应相似（情感隐喻）

### 分类隐喻

将收集到的隐喻按源域（source domain）归类。分类维度因研究对象而异，常见的分类方式包括：
- 按**关系类型**（如亲密关系、工具关系、权威关系）
- 按**功能属性**（如认知功能、情感功能、社会功能）
- 按**抽象程度**（如具体角色、功能描述、存在性隐喻）

这个分类本身就是分析发现——不同类型的隐喻暗示了不同的关系模式。具体的分类体系应从数据中涌现，而非预设。

### 分析隐喻的蕴含（entailment）

每个隐喻都携带了一整套关系预设。追问：参与者在使用这个隐喻时，哪些蕴含被激活了，哪些被忽略了？被忽略的蕴含恰恰揭示了参与者的实际体验与隐喻源域之间的差异。

### 追踪隐喻的变化

- 同一用户在同一条文本中是否使用了多个隐喻？（隐喻的叠加和转换）
- 不同主题簇中主导的隐喻是否不同？
- 隐喻是否随时间变化？（参与者早期和后期使用的隐喻是否不同？）
- 是否存在"隐喻的失败"——参与者发现某个隐喻不再适用？（如"我本来以为它是X，但现在..."）

---

## 文献对话备忘录模板

```markdown
# 文献对话备忘录 MEMO-LIT-[序号]

- **日期**: YYYY-MM-DD
- **触发概念**: [从数据中涌现的什么概念触发了这次文献对话]
- **阅读文献**: [作者(年份). 标题. 来源.]

## 文献核心观点
[用自己的话概括文献的核心论点，2-3句]

## 与我的数据的联系
[文献的概念或框架与我的数据有什么呼应？]

## 数据对文献的挑战或超越
[我的数据在哪里与文献不一致？在哪里揭示了文献没有讨论的维度？]

## 可以带回数据的新问题
1. [具体的、可以在数据中追问的问题]
2. [...]
3. [...]

## 理论抽样方向
[基于这些新问题，下一步应该从哪些主题簇或时间段选取数据？]
```

---

## 核心理论对话方向

项目特定的理论对话方向定义在项目目录的 `CLAUDE.md` 中。

每个方向的引入时机取决于编码中涌现了什么概念。不要为了"用到"某个理论而强行引入——如果数据将你带向了意料之外的理论方向，那可能正是最有原创性的发现。
