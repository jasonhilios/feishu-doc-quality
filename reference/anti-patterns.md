# 飞书文档反模式与修复案例

## 反模式 1：纯文本 FAQ

### 错误写法
```markdown
## 常见问题

Q：安装后会不会影响性能？
A：影响极小。

Q：数据存在哪里？
A：本地 SQLite。
```

**结果：** 所有 Q&A 被合并成一个 block，显示为一大段文字。

### 正确写法
```markdown
## 常见问题

<lark-table rows="3" cols="2" header-row="true" column-widths="300,430">
<lark-tr>
<lark-td>**问题**</lark-td>
<lark-td>**回答**</lark-td>
</lark-tr>
<lark-tr>
<lark-td>安装后会不会影响性能？</lark-td>
<lark-td>影响极小。</lark-td>
</lark-tr>
<lark-tr>
<lark-td>数据存在哪里？</lark-td>
<lark-td>本地 SQLite。</lark-td>
</lark-tr>
</lark-table>
```

---

## 反模式 2：表格后直接跟纯文本

### 错误写法
```markdown
| 配置项 | 默认值 |
|--------|--------|
| MODE   | code   |

如需中文模式，将 MODE 改为 code--zh。
```

**结果：** "如需中文模式..." 被合并到表格 block 中。

### 正确写法
```markdown
| 配置项 | 默认值 |
|--------|--------|
| MODE   | code   |

<callout emoji="bulb" background-color="light-blue">
如需中文模式，将 MODE 改为 code--zh。
</callout>
```

---

## 反模式 3：列表后无格式标记的总结段落

### 错误写法
```markdown
痛点：
- 上次修的 bug 又忘了
- 每次重新解释背景

claude-mem 就是来解决这个问题的。
```

**结果：** 总结句被合并到列表 block 中。

### 正确写法
```markdown
痛点：
- 上次修的 bug 又忘了
- 每次重新解释背景

**claude-mem 就是来解决这个问题的。** 它在后台自动记录每次工具调用。
```

---

## 修复操作模板

### 模式 A：删除合并内容，用表格重建

```bash
# 1. 删除合并的 block
feishu docx update <token> --mode delete --select "起始文本...结束文本"

# 2. 在正确位置插入表格
feishu docx update <token> --mode insert-before --select "下一个标题或区块" -c '<lark-table>...</lark-table>'
```

### 模式 B：替换为带格式的版本

```bash
feishu docx update <token> --mode replace --select "旧文本...旧结尾" -c '**关键词：** 新的段落内容'
```

### 模式 C：全文重写（最后手段）

```bash
# 1. 内容写入临时文件
# 2. 全量覆盖
feishu docx update <token> --mode overwrite --force -f /tmp/fixed-content.md
# 3. 验证
feishu fetch <token>
```
