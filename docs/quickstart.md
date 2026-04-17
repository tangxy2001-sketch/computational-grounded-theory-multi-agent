# 快速上手指南 / Quickstart Guide

## 前置条件

1. **Claude Code** — [安装指南](https://docs.anthropic.com/en/docs/claude-code)
2. **Git** — 用于克隆本仓库
3. （可选）**MCP Servers**：
   - `paper-search-mcp` — 学术论文搜索（支持 arXiv、PubMed、Google Scholar 等）
   - `mcp-obsidian` — Obsidian 知识库集成
   - `book-downloader` — 学术专著搜索与下载

## 安装步骤

### 1. 克隆仓库

```bash
git clone https://github.com/your-username/computational-grounded-theory-multi-agent.git
cd computational-grounded-theory-multi-agent
```

### 2. 安装全局 Skills

将四个技能目录复制到 Claude Code 的全局 skills 文件夹：

```bash
cp -r skills/grounded-theory ~/.claude/skills/
cp -r skills/gt-coding ~/.claude/skills/
cp -r skills/gt-literature ~/.claude/skills/
cp -r skills/gt-reflexivity ~/.claude/skills/
```

验证安装：打开 Claude Code，输入 `/skills`，应能看到四个已安装的 GT 技能。

### 3. 创建你的研究项目

```bash
mkdir ~/my-gt-research && cd ~/my-gt-research

# 复制项目级文件（Agent commands + BERTopic guide）
cp -r /path/to/toolkit/.claude .claude/

# 从模板创建项目协议
cp /path/to/toolkit/template/CLAUDE.md CLAUDE.md

# 创建项目目录结构
mkdir -p data/{raw,samples} \
         coding/{initial,focused,categories} \
         memos/{coding,theoretical,process,literature,reflexivity} \
         literature diagrams snapshots logs
```

### 4. 配置 CLAUDE.md

编辑 `CLAUDE.md`，填写你的研究信息：

```markdown
## 研究概况
本研究采用计算扎根理论框架，分析 [你的数据来源] 中 [你的研究现象]。

### 研究问题
- **RQ1**：[你的第一个研究问题]
- **RQ2**：[你的第二个研究问题]

### 编码焦点维度
- **[维度1]**：[说明]
- **[维度2]**：[说明]

### 研究者画像
研究者是 [定性研究初学者/有经验的研究者]。
```

### 5. 配置文献工作流（可选）

如果使用 Obsidian 作为知识库，在 `~/.claude/CLAUDE.md` 中添加：

```markdown
## Obsidian Vault
- Path: `/path/to/your/vault`
- Paper notes: `Papers/`
- PDF storage: `Resources/Papers/`
```

## 第一次会话示例

### 会话开始：评估状态

```
你: /main

Main Agent: 读取项目状态...
- 当前阶段：项目初始化
- 建议：从 BERTopic 结果中选取 3-5 个与研究问题最相关的主题簇
- 给 Coding Agent 的指令：浏览主题簇概况，选取代表性文本
```

### 核心工作：开始编码

```
你: /code

Coding Agent: 读取 Main Agent 的指令...
- 请提供 BERTopic 主题簇数据（topic_info.csv）
- 我们从浏览主题簇开始，选取与研究问题最相关的主题

你: [上传 topic_info.csv]

Coding Agent: 以下主题簇与你的研究问题关联度最高：
- Topic 25: [关键词] — 与 RQ1 相关
- Topic 42: [关键词] — 与 RQ2 相关
- ...
建议从 Topic 25 开始深度编码。每个主题簇选取 10-30 条代表性文本。

你: 好，从 Topic 25 开始

Coding Agent: 请提供 Topic 25 的代表性文本数据...
[开始逐条初始编码]
```

### 编码过程中

```
Coding Agent: 对这条文本的编码建议：
- 初始编码：[动名词形式的编码]
- 编码类型：过程编码
- 分析性说明：[为什么选这个编码]
- 替代编码：[其他可能]
这个编码是否捕捉到了你感受到的核心张力？

你: 我觉得不太对，这里更像是在 [你的理解]

Coding Agent: 你说得对，[讨论差异]...
[根据我们的讨论，我整理了一份备忘录草稿，你看看是否准确捕捉了你的想法]
```

### 会话结束：保存并生成快照

```
你: 今天先到这里

Coding Agent:
1. 编码表已保存到 coding/initial/initial_coding_topic25.xlsx
2. 备忘录已保存到 memos/coding/MEMO-IC-001.md
3. 对话日志已保存到 logs/session_2026-04-17_topic25_初始编码.md
4. 审计追踪已更新

你: /main

Main Agent: 生成进度快照...
- 当前阶段：初始编码（刚开始）
- 已编码：Topic 25，15条文本
- 涌现的概念：[列表]
- 下次建议：继续 Topic 25 的编码，关注 [特定方向]
快照已保存到 snapshots/snapshot_01_2026-04-17_初始编码开始.md
```

## 常见问题

### Q: 需要多少数据才能开始？
A: 计算发现层（BERTopic）需要较大规模数据。深度编码层从 3-5 个主题簇开始，每簇 10-30 条文本即可。

### Q: 三个 Agent 必须都用吗？
A: Main Agent 和 Coding Agent 是核心。Reflective Agent 在阶段转换时使用，如果你的研究不涉及方法论反思论文，可以跳过。

### Q: 可以不用 BERTopic 吗？
A: 可以。分层编码策略中的 Layer 1（计算发现）可以替换为其他主题建模方法或人工主题分类。核心的 Layer 2-4 不依赖 BERTopic。

### Q: Skills 必须安装为全局的吗？
A: 不必。你也可以将 `skills/` 目录复制到项目的 `.claude/skills/` 下作为项目级技能。全局安装的好处是跨项目复用。
