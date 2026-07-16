---
name: red-addwiki
version: 1.1.0
description: Use only when the user explicitly invokes $red-addwiki or says “调用 red-addwiki”. Extract exactly one durable, reusable knowledge point from the most recent relevant exchange in the current conversation and add it as one new row to the fixed Feishu Wiki Base table. Do not create Markdown files, Wiki child documents, or ordinary files, and do not trigger automatically for ordinary summaries, notes, Wiki questions, or knowledge-management requests.
---

# Red Add Wiki

仅在用户手动调用时执行。目标是把当前对话最近一次有复用价值的知识点，直接新增为飞书 Wiki Base 表中的一行记录。

## 固定目标

- Wiki/Base URL：`https://my.feishu.cn/wiki/Nr2IwD3pXimtnAkKTfQcl4mXn2b?table=tblXDvoqnio7NAlq&view=vew7pRbCBl`
- Base token：先用 `lark-cli base +url-resolve` 从 URL 解析；当前解析结果为 `Oev4bgZOfamfqes0QKYcew9SnKg`
- Table ID：`tblXDvoqnio7NAlq`
- 默认身份：`--as user`

这是 Base 表写入任务，不是 Wiki 节点创建任务。禁止使用 `wiki +node-create`、`docs +create`、`docs +update`、`markdown +create`，也不要在本地保存知识笔记文件。

## 工作流

1. 从当前对话中回看最近一次“用户问题 + 助手回答”形成的内容，提炼一个知识点。优先选择可迁移的方法、判断标准、命令模式、决策依据或概念；不要记录闲聊、临时状态或未经确认的推测。
2. 只记录一个知识点。若最近对话没有清晰、可复用的知识点，停止写入并请用户指定要记录的内容。
3. 用简洁中文拟定“知识点名称”，并准备“核心答案”“完整内容”“来源问题”。根据内容选择已有的“分类”和“标签”选项；无法确定时使用最接近的已有选项，不要擅自创建新选项。
4. 读取并遵循 `lark-shared` 和 `lark-base` 的相关说明。首次使用飞书用户身份时，先检查认证状态；需要授权时按 `lark-shared` 的 split-flow 规则处理，不要输出密钥。
5. 写入前用 `lark-cli base +field-list --base-token <base_token> --table-id tblXDvoqnio7NAlq --as user` 确认真实字段和选项。至少确认：`知识点名称`、`核心答案`、`完整内容`、`来源问题`、`来源Agent`；可选填写 `分类`、`标签`。
6. 用 `lark-cli base +record-search` 按 `知识点名称` 检查是否已有明显相同记录。若存在明确重复，停止写入并告知用户已有记录；默认不更新或覆盖既有行。
7. 无明确重复时，使用 `lark-cli base +record-upsert` 新增一条记录。JSON 至少包含：
   - `知识点名称`: 简洁标题
   - `核心答案`: 1-3 句话概括
   - `完整内容`: 自包含的知识说明
   - `来源问题`: 当前对话中的原始问题
   - `来源Agent`: `Codex`
   - `分类` / `标签`: 仅使用字段列表中已有的选项
8. 通过 `--json` 传入结构化记录；多行正文优先用安全的 JSON 编码方式生成，不要把 Markdown 文件作为中间载体。
9. 写入成功后用 `lark-cli base +record-get` 回读新记录，确认记录 ID、标题和核心字段都已保存。
10. 向用户报告：新增的表格记录 ID、知识点名称、使用的分类/标签和写入结果。失败时说明失败阶段和错误摘要，不要声称已写入。

## 记录内容建议

```json
{
  "知识点名称": "<简洁标题>",
  "核心答案": "<1-3句话的核心结论>",
  "完整内容": "<自包含的知识说明，可包含要点和注意事项>",
  "来源问题": "<当前对话中的原始问题>",
  "来源Agent": "Codex",
  "分类": "<已有分类>",
  "标签": ["<已有标签>"]
}
```

写作要求：忠实于当前对话，不补造事实；去掉访问令牌、密码和无关敏感信息；只写一个知识点；“完整内容”应能脱离当前对话独立理解；不要把本次执行动作本身伪装成长期知识。

## 当前已知字段

目标表当前包含：`分类`、`标签`、`完整内容`、`来源问题`、`来源Agent`、`关联知识点`、`知识点名称`、`核心答案`，以及系统字段“创建时间”“最后更新”。`关联知识点` 是链接字段，除非用户明确指定关联记录，否则不要填写。

当前已有分类包括：`学习方法`、`科学哲学`、`AI/技术`、`生活/效率`、`其他`、`运动/健身`。

当前已有标签包括：`费曼`、`学习`、`LLM`、`思维模型`、`方法论`、`心理学`、`效率`、`科学`、`AI`、`技术`、`学习方法`、`健身器械`、`EZ杆`、`曲杆`、`二头训练`、`肱三头训练`、`体育`、`足球`、`常识`。

## 安全与失败处理

- 这是用户明确授权后的写入操作，但范围仅限固定 Base 的固定表；不删除、不移动、不覆盖既有记录。
- 不要把 Wiki URL 直接当成 `--base-token`；先解析并使用返回的 `base_token`，表 ID 固定为 `tblXDvoqnio7NAlq`。
- 如果父 URL、Base、表或字段解析失败，停止，不换用其他 Base 或个人知识库重试。
- 如果认证或权限不足，按 `lark-shared` 报告授权/权限问题；不要静默切换到 bot。
- 如果新增记录成功但回读失败，仍报告已获得的记录 ID，并明确说明校验未完成。
