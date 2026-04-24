# feishu-doc-quality

Claude Code skill that prevents formatting issues when writing Feishu (飞书/Lark) documents via CLI.

## Problem

Feishu's Markdown Convert API has a known limitation: **consecutive plain-text paragraphs get merged into a single block** (soft line break / shift+enter instead of hard paragraph break). This causes FAQ sections, post-table notes, and list summaries to appear as one jumbled block instead of separate paragraphs.

## What this skill does

When triggered, it enforces **5 pre-write rules** and a **3-step post-write verification** flow:

| # | Rule | Fix |
|---|------|-----|
| 1 | FAQ / Q&A pairs | Use `<lark-table>` instead of plain text |
| 2 | Text after tables | Wrap in `<callout>` or add bold keywords |
| 3 | Summary after lists | Add bold keyword to first sentence |
| 4 | Consecutive paragraphs | Insert `---` or callout between them |
| 5 | Write method | Use `-f file.md` instead of inline `-c` |

After writing, it runs `feishu fetch` to verify and fixes any merged blocks.

## Install

One-line install (copies skill to your Claude Code skills directory):

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git /tmp/feishu-doc-quality-install && cp -r /tmp/feishu-doc-quality-install/SKILL.md /tmp/feishu-doc-quality-install/reference ~/.claude/skills/feishu-doc-quality/ && rm -rf /tmp/feishu-doc-quality-install && echo "Installed. Restart Claude Code to activate."
```

Or manually:

```bash
git clone https://github.com/jasonhilios/feishu-doc-quality.git
mkdir -p ~/.claude/skills/feishu-doc-quality
cp feishu-doc-quality/SKILL.md ~/.claude/skills/feishu-doc-quality/
cp -r feishu-doc-quality/reference ~/.claude/skills/feishu-doc-quality/
```

Restart Claude Code after installation. The skill auto-triggers whenever you write Feishu documents.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI, desktop, VS Code/JetBrains extension)
- [feishu CLI](https://github.com/nicholasxuu/feishu-cli) (`npm install -g @mi/feishu@latest`)

## How it works

The skill is a `SKILL.md` file that Claude Code auto-discovers from `~/.claude/skills/`. When you ask Claude to create or update a Feishu document, the skill loads and enforces the formatting rules before any `feishu docx` command is executed.

## File structure

```
feishu-doc-quality/
├── SKILL.md                    # Main skill definition
├── reference/
│   └── anti-patterns.md        # 3 anti-patterns with before/after examples
└── README.md
```

## License

MIT
