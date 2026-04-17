# 扎根理论研究项目 — 多Agent协作协议

## 研究概况

<!--
  在此填写你的研究概况。例如：
  本研究采用计算扎根理论（Nelson 2020）框架，分析[数据来源描述]中[研究对象描述]。
  数据已完成BERTopic主题建模（N个主题）。当前处于定性深度分析阶段。
-->

本研究采用计算扎根理论（Nelson 2020）框架，分析 [数据来源和规模] 中 [研究现象]。数据已完成BERTopic主题建模（N个主题）。当前处于定性深度分析阶段。

### 研究问题

<!-- 在此填写你的研究问题。建议2-3个，聚焦于意义建构和社会过程。 -->

- **RQ1**：[你的第一个研究问题]
- **RQ2**：[你的第二个研究问题]

### 编码焦点维度

<!--
  定义编码时重点关注的维度。这些维度应从研究问题推导而来。
  以下为示例维度，请替换为你自己的：
-->

- **[维度1名称]**：[简要说明，如：命名、功能赋予、情感投射]
- **[维度2名称]**：[简要说明]
- **[维度3名称]**：[简要说明]
- **人机关系隐喻**：概念隐喻系统、隐喻间的张力
- **平台化表达**：表演性、话语策略、社区互动

### 核心理论对话方向

<!--
  列出你的研究可能对话的理论传统。不需要预先确定——
  随着编码推进，理论方向会从数据中涌现。以下为示例：
-->

- [理论方向1，如：技术的社会建构论（SCOT, Pinch & Bijker）]
- [理论方向2，如：概念隐喻理论（Lakoff & Johnson）]
- [理论方向3]
- [理论方向4]

### 研究者画像

<!--
  描述研究者的背景，帮助Claude校准互动方式。例如：
  - 研究者是定性研究初学者 → Claude会在每步操作前解释为什么
  - 研究者是资深研究者 → Claude减少方法论解释，更注重对话
-->

研究者是 [经验水平] 的定性研究者。[根据需要添加说明，如：每一步分析操作前简要解释为什么做这一步。使用方法论术语时附带简短解释。]

## 三个Agent

本项目使用三个Agent协作，通过自定义命令切换：

| Agent | 命令 | 核心职责 | 技能依赖 | 输出目录 |
|-------|------|----------|----------|----------|
| **Main Agent** | `/main` | 方法论守护、阶段判断、进度追踪 | grounded-theory | `snapshots/`, `audit_trail.xlsx` |
| **Coding Agent** | `/code` | 编码、文献对话、备忘录、理论建构 | gt-coding, gt-literature | `coding/`, `memos/coding\|theoretical\|literature\|process` |
| **Reflective Agent** | `/reflect` | 人机协作反思 | gt-reflexivity | `memos/reflexivity/` |

### 典型会话流程

```
/main  → 评估进度，生成本次会话的工作指令
  ↓
/code  → 执行编码/文献/备忘录（主要工作时间）
  ↓
/main  → 阶段性检查（可选，长会话中期做一次）
  ↓
/code  → 继续工作，会话结束前保存日志和更新审计追踪
  ↓
/reflect → 阶段转换时生成反思性备忘录（不是每次会话都需要）
  ↓
/main  → 会话结束，生成 snapshot
```

## Agent交接协议

### Main → Coding
- Main Agent 的 snapshot 包含「给Coding Agent的指令」段落，列出具体任务
- Coding Agent 每次会话开始时读取最新 snapshot

### Coding → Main
- Coding Agent 每次会话结束前更新 `audit_trail.xlsx`（追加当次记录）
- Main Agent 审查 `coding/`、`memos/`、`audit_trail.xlsx`

### Coding → Reflective
- **硬性规则**：Coding Agent 每次会话结束前必须保存对话日志到 `logs/session_YYYY-MM-DD_描述.md`
- 对话日志记录关键编码讨论、分歧、决策，不是逐字记录
- Reflective Agent 依赖 `logs/`、`coding/`、`memos/` 作为反思素材

### Main → Reflective
- Reflective Agent 读取 `snapshots/` 了解当前阶段
- Main Agent 在 snapshot 中标注是否到了需要反思的阶段转换节点

## 文件组织

```
your-project/
├── CLAUDE.md               ← 本文件（项目协议）
├── data/
│   ├── raw/                # BERTopic结果（topic_info.csv, topics_over_time.csv, doc_topics.csv）
│   └── samples/            # 小批量编码数据（按主题簇导出的CSV）
├── coding/
│   ├── initial/            # 初始编码表（Excel）
│   ├── focused/            # 聚焦编码表（Excel）
│   └── categories/         # 概念范畴表（Excel）
├── memos/
│   ├── coding/             # 编码备忘录 MEMO-IC-XXX.md
│   ├── theoretical/        # 理论备忘录 MEMO-TC-XXX.md
│   ├── process/            # 过程备忘录 MEMO-PC-XXX.md
│   ├── literature/         # 文献对话备忘录 MEMO-LIT-XXX.md
│   └── reflexivity/        # 反思性备忘录 REFLEX-XXX.md
├── literature/             # 文献建议清单
├── diagrams/               # 概念关系图（Mermaid/HTML）
├── snapshots/              # 研究进度快照
├── logs/                   # 会话对话日志
└── audit_trail.xlsx        # 分析追踪表（Main Agent维护）
```

### 命名规则

- 编码表：`{类型}_coding_{主题}.xlsx`（如 `initial_coding_topic25.xlsx`），同一主题的编码写在同一文件中
- 编码备忘录：`MEMO-IC-{序号}.md`
- 理论备忘录：`MEMO-TC-{序号}.md`
- 过程备忘录：`MEMO-PC-{序号}.md`
- 文献备忘录：`MEMO-LIT-{序号}.md`
- 反思备忘录：`REFLEX-{序号}_{阶段描述}.md`
- 快照：`snapshot_{序号}_{日期}_{描述}.md`
- 日志：`session_{日期}_{描述}.md`

## 所有Agent的通用规则

1. **数据为锚**：所有概念和编码必须能回溯到具体数据片段
2. **区分描述与解读**：在备忘录中明确标记「数据在说什么」与「我的初步解读（待研究者审核）」
3. **标记证据强度**：注明编码/概念基于多少篇文档（如"4篇可追溯案例"）
4. **保持反思性**：经常问「为什么做出这个分析选择？」
5. **研究者决定权**：所有概念命名、理论判断的最终决定权在研究者
6. **文件优先**：所有分析产出必须保存为文件，不能仅停留在对话中
7. **in vivo优先**：编码命名尽量保留用户原话，避免过早用学术语言覆盖
