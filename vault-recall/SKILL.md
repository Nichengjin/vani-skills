---
name: vault-recall
description: "使用 vault CLI 检索跨 Agent 历史会话，并按需读取精简 conversation-json 或 Markdown。当用户提到过去的对话、上次的决定、我们之前聊过、find the session where，或要继续当前上下文之外的历史工作时使用。"
---

# Vault Recall

用 `vault` CLI 按需定位和读取 Claude Code、ClawsGO、Codex、Grok、Kimi、Cursor、Devin CLI、Amp 与
Vani ACP 历史会话。通过 CLI 消费来源适配器和可重建缓存，不直接读取完整原始日志、
Canonical 存档或内部数据库，也不维护另一份长期 Markdown 库。

## 工作流:先定位,再读取

1. **定位** — 二选一:
   - 用户说"最近/上次":`vault recent [PATH] --limit 10 --json`。
   - 用户给了主题/关键词:`vault search <keyword> [PATH] --limit 10 --snippets 3 --json`。
2. **读取** — 从结果取得 `agent`、`sessionId`（缺失时使用 `sourceId`）和项目路径，默认请求精简机器格式:

   ```sh
   vault show <sessionId> [PATH] --agent <agent> --format conversation-json
   ```

   使用结果中的完整 Agent ID（例如 `openai-codex`、`claude-code`），不要猜来源。
   session id 支持唯一前缀，但已知完整 ID 时优先使用完整值；`--agent` 避免跨来源 ID 歧义。
3. **回答** — 根据相关消息与工具结果回答，标明引用的会话身份；区分当时的决定与当前仍待验证的状态。

`PATH` 默认当前目录；跨项目时传目标工作区路径。含空格的路径和关键词须正确引用。
cloud 默认关闭；需要读取 Amp 云会话时，在定位和读取命令中显式加 `--with-cloud`。
`show` 仅获取选中的云会话正文，不批量下载其他会话。

## 输出格式

- `recent --json` 返回 `schemaVersion`、`total`、`sessions`；`search --json` 返回
  `results`，每项含 `session` 和 `snippets`（含 `eventId`、`blockIndex`）。先利用片段缩小范围。
- **默认正文：`conversation-json`**。校验 `schemaVersion=1`、`kind=conversation`、
  `sourceSchemaVersion=1`、`projection=conversation-v1`，读取 `session/events/toolInvocations`。
  保留消息、工具输入输出、图片引用、事件和工具关联及未知内容；仅省略已审计的存档副本。
  当前瘦身 Codex 存档副本与 Devin 的 `rawStep`，其他来源未审计的扩展保守透传。这不是摘要，也没有自动截断正文。
- **纯文字阅读：`--format markdown`**。只需人类可读对话或用户明确要求 Markdown 时使用；
  不把它当完整事件图或完整工具细节。
- **完整存档：`--format canonical-json`**。只在明确需要完整 provenance、存档或原生回写时请求。
  `conversation-json` 是只读派生视图，不能作为完整 Canonical 写回 store 或交给原生回写接口。

## 在回复里引用会话

回答里提到某次历史会话时，附上它的会话引用，Vani 会把它渲染成可点击的会话卡片（其他终端里就是一段可复制的 ID）：

- 格式：`vani:<agent>:<sessionId>`，`agent` 与 `sessionId` 原样取自 `recent` / `search` 结果（例如
  `vani:claude-code:47aa1e6b-…`）。不要自己改写 agent id，不要只写前缀。
- 想要卡片就让引用单独占一行（或一个列表项只放引用）；写在句子里会变成行内 chip。用反引号包住也可以。
- 不要放进代码块，代码块里保持原文不渲染。也不要写成 Markdown 链接 `[标题](vani:...)`。
- 当前这条会话不用引用。

## 注意事项

- **结果可能包含当前正在进行的这个 session**(时间最新、内容与本轮对话雷同的那条)。
  引用"过去的对话"时跳过它。
- 搜索只覆盖 user/assistant 的文本,不覆盖工具调用输出与 thinking。搜报错信息等
  工具产物可能搜不到;换成当时讨论用的自然语言关键词再试。
- 命中过多先细化关键词，不盲目增大 `--limit`。精简输出仍可能很大；需要筛选时，在工具端
  完整解析 JSON 后按命中的 `eventId` 选取相关事件及必要上下文，再按 `toolInvocationId`
  取关联工具。不要把单行 JSON 用 `head`/`grep` 截断后当有效结果，也不要一次注入全部无关正文。
- 旧 Vault 明确不支持 `conversation-json` 时提示升级；若任务仅需文字，可显式改读 Markdown。
  其他错误按真实原因处理，不把超时、超限或来源读取失败当作版本问题，亦不自动重试完整 Canonical。
- 关键词是大小写不敏感的子串匹配,不支持正则;中文关键词可直接使用。
