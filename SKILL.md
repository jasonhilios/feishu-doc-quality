---
name: feishu-doc-quality
description: |-
  飞书文档写入质量保障。在执行 feishu docx create、feishu docx update 写入飞书文档时自动触发，确保格式正确、段落独立、无合并问题。
  触发条件（满足任一激活）：
  1. 正在执行或即将执行 feishu docx create / feishu docx update 写入文档内容；
  2. 用户要求创建飞书文档、写入飞书文档、更新飞书文档；
  3. 用户提到"写文档""创建文档""更新文档"且目标是飞书。
  此 skill 优先级高于默认写入行为 — 必须先按本 skill 的规则组织内容，再调用 feishu CLI。
version: 1.0.0
---

# 飞书文档写入质量保障

## 核心问题

飞书 Markdown Convert API 存在已知限制：**连续的纯文字段落会被合并成单个 block（shift+enter 软换行），而非独立段落。** 本 skill 提供写入前的内容组织规则和写入后的验证修复流程。

---

## 写入前：内容组织规则

### 规则 1：连续 Q&A / FAQ 必须用 lark-table

禁止用纯文本段落写连续的问答对。必须转为表格：

```html
<lark-table rows="N" cols="2" header-row="true" column-widths="300,430">
<lark-tr>
<lark-td>**问题**</lark-td>
<lark-td>**回答**</lark-td>
</lark-tr>
<lark-tr>
<lark-td>问题内容</lark-td>
<lark-td>回答内容</lark-td>
</lark-tr>
</lark-table>
```

### 规则 2：表格后紧跟的说明文字必须独立

表格后的补充说明不能直接跟在表格后面写纯文本。两种方案：

**方案 A**（推荐）：用 callout 包裹
```html
<callout emoji="bulb" background-color="light-blue">
补充说明内容
</callout>
```

**方案 B**：在说明文字中加入格式标记（如粗体关键词），使其被识别为独立 block。

### 规则 3：列表后的总结段落需要格式标记

列表后的总结性段落容易被合并。解决方案：给段落首句加粗体关键词。

错误写法：
```
- 要点 1
- 要点 2
总结内容在这里...
```

正确写法：
```
- 要点 1
- 要点 2

**总结：** 内容在这里...
```

### 规则 4：连续纯文段落之间用分隔元素隔开

如果文档中有 3 个以上连续的纯文段落，用以下方式分隔：
- 在段落间插入 `---` 分隔线
- 或将重点段落用 callout 包裹
- 或用标题将内容分组

### 规则 5：写入方式选择

| 场景 | 推荐方式 |
|------|----------|
| 创建新文档 | `feishu docx create "标题" -f /tmp/content.md` |
| 全量替换 | `feishu docx update <token> --mode overwrite --force -f /tmp/content.md` |
| 局部修改 | `feishu docx update <token> --mode replace --select "起始...结束" -c "新内容"` |
| 追加内容 | `feishu docx update <token> --mode append -f /tmp/content.md` |

**关键：** 优先用 `-f file.md` 而非 `-c "inline content"`。内容先写入临时文件再传入。

---

## 写入后：验证与修复流程

### 步骤 1：验证

写入后立即执行：
```bash
feishu fetch <doc_token_or_url>
```

检查 markdown 输出中是否存在合并问题：
- 连续行之间无空行分隔 → 可能被合并
- 同一 block 内出现多段内容 → 确认被合并

### 步骤 2：修复

对已合并的内容，使用定向修复：

**删除并重新插入：**
```bash
feishu docx update <token> --mode delete --select "起始文本...结束文本"
feishu docx update <token> --mode insert-before --select "下一个区块的起始文本" -c '修复后的内容'
```

**替换为表格：**
```bash
feishu docx update <token> --mode replace --select "起始...结束" -c '<lark-table>...</lark-table>'
```

### 步骤 3：注意频率限制

飞书 API 频率限制约为 5 次/分钟。连续操作时：
- 每次操作后检查返回值
- 遇到 `rate_limited` 错误时等待重试
- 尽量减少操作次数，一次性写入完整内容

---

## 速查清单

写入前对照检查：

- [ ] FAQ/Q&A 内容是否用了 lark-table？
- [ ] 表格后的说明文字是否独立成块（callout 或带格式标记）？
- [ ] 列表后的总结段落是否加了粗体关键词？
- [ ] 是否有 3+ 连续纯文段落需要分隔？
- [ ] 内容是否写入了临时文件再用 `-f` 传入？
- [ ] 写入后是否执行了 `feishu fetch` 验证？
