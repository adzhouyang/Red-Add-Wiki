---
name: red-addwiki
version: 1.0.0
description: Use only when the user explicitly invokes $red-addwiki or says “调用 red-addwiki”. Extract exactly one durable, reusable knowledge point from the most recent relevant exchange in the current conversation and record it as a new Feishu Wiki document under the fixed parent Wiki URL https://my.feishu.cn/wiki/Nr2IwD3pXimtnAkKTfQcl4mXn2b?table=tblXDvoqnio7NAlq&view=vew7pRbCBl. Do not trigger automatically for ordinary summaries, notes, Wiki questions, or knowledge-management requests.
---

# Red Add Wiki

仅在用户手动调用时执行。目标是把当前对话最近一次有复用价值的知识点，写成一篇简洁的中文 Wiki 子文档。

## 固定目标

父节点固定为：

`https://my.feishu.cn/wiki/Nr2IwD3pXimtnAkKTfQcl4mXn2b?table=tblXDvoqnio7NAlq&view=vew7pRbCBl`

优先使用用户身份 `--as user`。不要把该 URL 当作普通 Drive 文件夹，也不要写入其他个人知识库或默认 Wiki 空间。

## 工作流

1. 从当前对话中回看最近一次“用户问题 + 助手回答”形成的内容，提炼一个知识点。优先选择可迁移的方法、判断标准、命令模式、决策依据或概念；不要记录闲聊、临时状态或未经确认的推测。
2. 只记录一个知识点。若最近对话没有清晰、可复用的知识点，停止写入并请用户指定要记录的内容。
3. 用简洁中文拟定标题，避免标题含日期、序号或无意义的“学习笔记”。
4. 读取并遵循 `lark-shared`、`lark-wiki`、`lark-doc` 的相关说明。首次使用飞书用户身份时，先检查认证状态；需要授权时按 `lark-shared` 的 split-flow 规则处理，不要输出密钥。
5. 用 `lark-cli wiki +node-list` 查看固定父节点下的现有子节点，检查是否已有相同或明显重复的标题/主题。不要为了查重全文读取所有文档。
6. 如发现明确相同的既有文档，先停止并告知用户已有文档标题和链接，请用户确认是否覆盖/合并；本 Skill 默认不修改既有文档。
7. 如无明确重复，在固定父节点下创建新的 `docx` Wiki 子节点。使用 `lark-cli wiki +node-create --parent-node-token <固定URL> --title "<标题>" --as user`，不要直接猜 space_id。
8. 创建成功后，使用 `lark-cli docs +update --doc <obj_token或文档URL> --command overwrite --doc-format markdown --content @<当前工作目录下的相对Markdown文件>` 写入正文。正文短小，一次写完；不要把临时 Markdown 文件留在用户项目目录。
9. 写入完成后重新读取/校验文档标题和正文，确认节点确实位于固定父节点下。
10. 向用户报告：知识点标题、Wiki 文档 URL、是否检测到重复；失败时说明失败阶段和原始错误摘要，不要声称已写入。

## 正文格式

```markdown
# <知识点标题>

## 摘要
<用 1-3 句话说明核心结论或方法。>

## 要点
- <关键点 1>
- <关键点 2>
- <关键点 3，可选>

## 适用场景
<说明什么时候有用，以及一个必要的限制或注意事项。>

## 来源上下文
<一句话说明它来自当前对话的哪个问题。>
```

写作要求：忠实于当前对话，不补造事实；去掉个人敏感信息、访问令牌和无关上下文；标题和正文保持自包含；不要把“本次已执行的动作”伪装成长期知识。

## 安全与失败处理

- 这是用户明确授权后的写入操作，但仍只写固定 Wiki 父节点；不删除、不移动、不覆盖既有文档。
- URL 中的 `table` / `view` 查询参数属于目标链接的一部分，保留原样传给 CLI，让 CLI 解析 Wiki 节点。
- 如果父节点解析失败、权限不足或认证缺失，停止，不要换用 bot 或其他目标重试；按 `lark-shared` 报告授权/权限问题。
- 如果节点创建成功但正文写入失败，报告已创建的空节点 URL，便于用户处理；不要重复创建多个节点。
