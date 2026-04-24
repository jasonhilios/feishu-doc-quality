<div align="center">

# feishu-doc-quality

**Claude Code skill — 解决 AI 写飞书文档时的格式错乱问题**

*Prevent formatting issues when AI writes Feishu/Lark documents via CLI*

[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-7C3AED?style=flat-square&logo=anthropic)](https://docs.anthropic.com/en/docs/claude-code)
[![Lark CLI](https://img.shields.io/badge/Lark%20CLI-Compatible-3370FF?style=flat-square&logo=lark)](https://github.com/larksuite/cli)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

[中文](#中文) · [English](#english)

</div>

---

<a id="中文"></a>

## 中文

### 这个 skill 解决什么问题？

当 AI 工具（Claude Code、Cursor 等）通过 CLI 写入飞书文档时，**飞书 Markdown Convert API** 有一个关键限制：

> **连续的纯文字段落会被合并成单个 block**（shift+enter 软换行），而非独立的段落 block。

**实际效果：**

| 你写的内容 | 飞书实际显示 |
|---|---|
| 4 组独立的 Q&A | 一大坨文字糊在一起 |
| 表格 + 补充说明 | 说明文字被塞进表格 block 里 |
| 列表 + 总结段落 | 总结粘在最后一个列表项上 |
| 多个独立段落 | 一段文字里出现诡异的换行 |

### 修复前后对比

**修复前**（不用本 skill）— FAQ 用纯文本写入，全部合并成一个 block：

```
Q：安装后会影响性能吗？A：影响极小。Q：数据存在哪里？A：本地 SQLite。
Q：支持哪些 IDE？A：Claude Code、Cursor 等。
```

**修复后**（用本 skill）— FAQ 转为结构化表格，每一行清晰独立：

| 问题 | 回答 |
|---|---|
| 安装后会影响性能吗？ | 影响极小。 |
| 数据存在哪里？ | 本地 SQLite。 |
| 支持哪些 IDE？ | Claude Code、Cursor 等。 |

### 功能说明

这是一个 [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code#skills)，在写入飞书文档时自动激活，强制执行 **5 条写入前规则** + **写入后验证流程**。

**5 条写入前规则：**

| # | 受影响模式 | 解决方案 |
|---|---------|----------|
| 1 | 连续 Q&A / FAQ | 转为 `<lark-table>` |
| 2 | 表格后紧跟说明文字 | 用 `<callout>` 包裹或加粗体关键词 |
| 3 | 列表后的总结段落 | 首句加粗体关键词 |
| 4 | 连续纯文段落 | 插入 `---` 分隔线或 callout |
| 5 | 写入方式 | 用 `-f file.md` 而非内联 `-c` |

**写入后验证流程：**

1. 执行 `feishu fetch` 检查实际 block 结构
2. 检测合并问题（连续行之间无空行分隔）
3. 用定向 `replace` / `delete` + `insert` 命令修复

### 安装

**前置依赖：**

| 依赖 | 用途 | 安装方式 |
|---|---|---|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AI 编程助手 | 参见 Anthropic 文档 |
| [Lark CLI](https://github.com/larksuite/cli) | 飞书文档操作 | `npm install -g @larksuite/cli` |

> 本 skill 基于飞书扩展 Markdown 语法（`lark-table`、`callout` 等），适用于所有使用该语法的飞书 CLI 工具。

**一行命令安装 skill：**

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git /tmp/feishu-doc-quality-install && \
mkdir -p ~/.claude/skills/feishu-doc-quality && \
cp /tmp/feishu-doc-quality-install/SKILL.md ~/.claude/skills/feishu-doc-quality/ && \
cp -r /tmp/feishu-doc-quality-install/reference ~/.claude/skills/feishu-doc-quality/ && \
rm -rf /tmp/feishu-doc-quality-install && \
echo "安装完成，重启 Claude Code 即可生效。"
```

**手动安装：**

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git
mkdir -p ~/.claude/skills/feishu-doc-quality
cp feishu-doc-quality/SKILL.md ~/.claude/skills/feishu-doc-quality/
cp -r feishu-doc-quality/reference ~/.claude/skills/feishu-doc-quality/
```

安装后**重启 Claude Code**。Skill 从 `~/.claude/skills/` 自动发现，每次写飞书文档时自动触发。

### 工作原理

```
用户："创建一个飞书文档，包含 FAQ 和配置表"
                    │
                    ▼
        ┌───────────────────────┐
        │  feishu-doc-quality   │  ← Skill 自动触发
        │  (加载 SKILL.md)      │
        └───────────┬───────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   写入前规则            内容生成
   (应用 5 条规则)       (组织 markdown)
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
         feishu docx create -f /tmp/doc.md
                    │
                    ▼
          写入后验证
          feishu fetch → 检查 block 结构
                    │
              ┌─────┴─────┐
              ▼           ▼
           格式正确 ✓   修复合并问题
                       (replace / insert)
```

### 文件结构

```
~/.claude/skills/feishu-doc-quality/
├── SKILL.md                     # 主文件（自动加载）
└── reference/
    └── anti-patterns.md         # 3 个反案例（错误写法 vs 正确写法）
```

### 常见问题

**Q：这个 skill 能用在 Cursor 或其他 AI 工具上吗？**

A：这是一个 Claude Code skill，设计用于 Claude Code。如果其他工具兼容 Claude Code 的 skill 格式，也可能支持。

**Q：会影响文档写入速度吗？**

A：不会。规则在内容生成阶段应用（写入前），写入后验证只多一次 `feishu fetch` 调用，耗时约 1 秒。

**Q：可以自定义规则吗？**

A：可以。直接编辑 `~/.claude/skills/feishu-doc-quality/SKILL.md`，规则在"写入前：内容组织规则"部分。

### 相关链接

- [Lark CLI](https://github.com/larksuite/cli) — 飞书/Lark 官方命令行工具（200+ 命令，22 个 AI Agent Skills）
- [Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code#skills) — Claude Code Skill 官方文档
- [飞书开放平台](https://open.feishu.cn/) — 飞书 API 文档
- [飞书扩展 Markdown 语法](https://open.feishu.cn/document/client-docs/docs-add-on/extended-markdown) — lark-table、callout 等语法参考

---

<a id="english"></a>

## English

### The Problem

When AI tools (Claude Code, Cursor, etc.) write documents to Feishu/Lark via CLI, the **Feishu Markdown Convert API** has a critical limitation:

> **Consecutive plain-text paragraphs are merged into a single block** (soft line break / shift+enter), instead of being separate paragraph blocks.

**What happens:**

| What you write | What Feishu actually shows |
|---|---|
| 4 separate Q&A pairs | 1 giant wall of text |
| Table + caption paragraph | Caption stuck inside the table block |
| Bullet list + summary | Summary glued to the last list item |
| Multiple paragraphs | One blob with weird line breaks |

### Before & After

**Before** (without this skill) — FAQ as plain text, all merged into one block:

```
Q: Will this affect performance? A: Minimal impact. Q: Where is data stored?
A: Local SQLite. Q: Which IDEs? A: Claude Code, Cursor, etc.
```

**After** (with this skill) — FAQ as a structured table, each row is clean:

| Question | Answer |
|---|---|
| Will this affect performance? | Minimal impact. |
| Where is data stored? | Local SQLite. |
| Which IDEs? | Claude Code, Cursor, etc. |

### What This Skill Does

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code#skills) that automatically activates when writing Feishu documents. It enforces **5 pre-write rules** and a **post-write verification** flow.

**5 Pre-write Rules:**

| # | Pattern | Solution |
|---|---------|----------|
| 1 | FAQ / Q&A pairs | Convert to `<lark-table>` |
| 2 | Text after tables | Wrap in `<callout>` or add bold keywords |
| 3 | Summary after lists | Add bold keyword to first sentence |
| 4 | Consecutive plain paragraphs | Insert `---` separators or callouts |
| 5 | Write method | Use `-f file.md` instead of inline `-c` |

**Post-write Verification:**

1. Run `feishu fetch` to inspect the actual block structure
2. Detect merged blocks (consecutive lines without blank-line separation)
3. Fix with targeted `replace` / `delete` + `insert` commands

### Install

**Prerequisites:**

| Dependency | Purpose | Install |
|---|---|---|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AI coding assistant | See Anthropic docs |
| [Lark CLI](https://github.com/larksuite/cli) | Feishu/Lark document operations | `npm install -g @larksuite/cli` |

> This skill is based on Feishu extended Markdown syntax (`lark-table`, `callout`, etc.) and works with any Feishu CLI that uses this syntax.

**One-line install:**

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git /tmp/feishu-doc-quality-install && \
mkdir -p ~/.claude/skills/feishu-doc-quality && \
cp /tmp/feishu-doc-quality-install/SKILL.md ~/.claude/skills/feishu-doc-quality/ && \
cp -r /tmp/feishu-doc-quality-install/reference ~/.claude/skills/feishu-doc-quality/ && \
rm -rf /tmp/feishu-doc-quality-install && \
echo "Done. Restart Claude Code to activate."
```

**Manual install:**

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git
mkdir -p ~/.claude/skills/feishu-doc-quality
cp feishu-doc-quality/SKILL.md ~/.claude/skills/feishu-doc-quality/
cp -r feishu-doc-quality/reference ~/.claude/skills/feishu-doc-quality/
```

Then **restart Claude Code**. The skill auto-discovers from `~/.claude/skills/` and triggers whenever you write Feishu documents.

### How It Works

```
User: "Create a Feishu doc with FAQ and config table"
                    │
                    ▼
        ┌───────────────────────┐
        │  feishu-doc-quality   │  ← Skill auto-triggers
        │  (SKILL.md loaded)    │
        └───────────┬───────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Pre-write rules      Content generation
   (5 rules applied)    (organize markdown)
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
         feishu docx create -f /tmp/doc.md
                    │
                    ▼
          Post-write verification
          feishu fetch → check blocks
                    │
              ┌─────┴─────┐
              ▼           ▼
           Clean ✓    Fix merged blocks
                      (replace/insert)
```

### File Structure

```
~/.claude/skills/feishu-doc-quality/
├── SKILL.md                     # Main skill definition (auto-loaded)
└── reference/
    └── anti-patterns.md         # 3 anti-patterns (wrong vs correct examples)
```

### FAQ

**Q: Does this work with Cursor or other AI tools?**

A: This is a Claude Code skill. It may work with other tools that support the Claude Code skill format.

**Q: Will it slow down document writing?**

A: No. The rules are applied during content generation. Post-write verification adds one `feishu fetch` call (~1 second).

**Q: Can I customize the rules?**

A: Yes. Edit `~/.claude/skills/feishu-doc-quality/SKILL.md` directly.

### Related Links

- [Lark CLI](https://github.com/larksuite/cli) — Official Lark/Feishu CLI (200+ commands, 22 AI Agent Skills)
- [Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code#skills) — Claude Code Skill documentation
- [Feishu Open Platform](https://open.feishu.cn/) — Feishu API documentation
- [Feishu Extended Markdown](https://open.feishu.cn/document/client-docs/docs-add-on/extended-markdown) — lark-table, callout syntax reference

---

## Contributing

Contributions welcome. Open an issue or submit a PR.

## License

[MIT](LICENSE)
