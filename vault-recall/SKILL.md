---
name: vault-recall
description: Recall past Claude Code conversations with the `vault` CLI — list recent sessions, keyword-search conversation history, and read a full past session as Markdown. Use when the user references a previous conversation/session/chat ("我们之前聊过"、"上次那个对话"、"find the session where..."), asks what was discussed or decided earlier, or wants to continue work from an earlier session that is not in the current context.
---

# Vault Recall

用 `vault` CLI 按需检索历史 Claude Code 对话。一切都是无状态的:搜索直接读
`~/.claude/projects/` 下的 JSONL 事实源,永远新鲜;Markdown 只在读取时按需渲染,
不产生任何需要维护的文件。

## 工作流:先定位,再读取

1. **定位** — 二选一:
   - 用户说"最近/上次":`vault recent [PATH] [--limit N]`(默认 10 条,最新在前)。
   - 用户给了主题/关键词:`vault search <keyword> [PATH] [--limit N] [--snippets M]`。
2. **读取** — 拿到 session id 后:`vault show <sessionId> [PATH]`,
   完整对话以 Human Markdown 打到 stdout。id 支持唯一前缀(如 `vault show 305ea2c8`)。

`PATH` 默认当前目录,语义与 `vault export` 一致:可以传工作区路径(自动映射到
`~/.claude/projects/<slug>`),也可以直接传 JSONL 所在目录。跨项目回忆时把那个
项目的工作区路径作为 PATH 传入。

## 输出格式

`recent` 与 `search` 每个 session 一行:`<sessionId>  <时间>  <标题>`;
`search` 在行下缩进列出 `[user]` / `[assistant]` 命中片段。

## 注意事项

- **结果可能包含当前正在进行的这个 session**(时间最新、内容与本轮对话雷同的那条)。
  引用"过去的对话"时跳过它。
- 搜索只覆盖 user/assistant 的文本,不覆盖工具调用输出与 thinking。搜报错信息等
  工具产物可能搜不到;换成当时讨论用的自然语言关键词再试。
- 命中过多时先细化关键词,而不是加大 `--limit`;`vault show` 输出的是整段对话,
  很长的 session 酌情用 `head`/`grep` 截取相关部分,避免撑爆上下文。
- 关键词是大小写不敏感的子串匹配,不支持正则;中文关键词可直接使用。
