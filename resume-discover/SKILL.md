---
name: resume-discover
description: "使用 resume CLI 发现用户的项目目录、最近活跃项目、项目分组和关联 Agent 会话。当用户问有哪些项目、某个项目在哪、找项目目录、最近在做什么项目，或需要先定位工作区再继续历史工作时使用。"
---

# Resume Discover

用 `resume` CLI 从本机会话索引定位真实项目 `cwd`，再按需查询项目会话。项目目录发现用 Resume，历史对话正文检索用 `vault-recall`。

## 工作流：先定位项目，再查询会话

1. **检查入口**：用 `command -v resume` 确认 CLI 可用；需要核对当前版本支持的命令时运行 `resume help`。缺少 CLI 时说明需要先安装 Vani Resume。
2. **列出项目**：运行 `resume projects`。结果按最后活跃时间降序排列，字段为 Tab 分隔：

   ```text
   LAST_ACTIVE  SESSIONS  AGENTS  CWD
   ```

   `LAST_ACTIVE` 是 Unix 秒，`SESSIONS` 是索引会话数，`AGENTS` 是逗号分隔的来源，`CWD` 是项目绝对路径。按 Tab 解析，保留含空格路径；不要按任意空白拆列。回答活跃时间时转换到用户时区。
3. **缩小范围**：按用户给出的项目名或路径片段匹配 `CWD`；用户问最近项目时取前几项。匹配示例：

   ```sh
   resume projects | awk -F '\t' 'NR == 1 || index(tolower($4), "project-keyword")'
   ```

   将示例关键词替换为本次目标；把用户输入作为数据传给筛选工具，不拼接成待执行的 shell 代码。同名目录用完整路径、活跃时间和来源区分；仍有多个候选时展示候选供用户选择，不直接进入任意一个。
4. **按需刷新**：索引为空、没找到目标，或用户要求最新项目时，运行一次 `resume scan`，再查询 `resume projects`。扫描读取本地来源并更新派生索引，不修改源会话。保留并说明来源 warning，报告哪些来源未刷新成功；不要重复扫描来掩盖来源错误。
5. **确认目录**：要在项目里执行工作前，用 `test -d "/absolute/project/path"` 检查目录存在，并把所选 `cwd` 作为工具的工作目录。历史记录里的路径可能已迁移或删除；不存在时如实说明，按用户指定的父目录进一步查找。
6. **按需查看会话**：从项目结果复制完整 `CWD`，正确引用路径：

   ```sh
   resume sessions "/absolute/project/path"
   ```

   输出字段为 `UPDATED_AT`、`AGENT`、`SESSION_ID`、`TITLE`、`SOURCE`，按更新时间降序排列。先用标题和时间筛选相关会话，只返回与任务相关的少量结果。

## 项目分组

用户按产品或项目集合查找时，先运行：

```sh
resume group list
resume group sessions "group-id"
```

`group list` 返回 `ID`、`NAME`、`MEMBERS`（成员数量）。从结果复制 group ID，再用 `group sessions` 查看成员项目下的会话；其中的 `CWD` 指向真实项目目录。分组是用户手动维护的逻辑集合，不是可用作工作目录的文件夹。

## 与 Vault 衔接

- 用户只问项目在哪：返回匹配的绝对路径、必要的区分依据即可，不读取历史正文。
- 用户要找过去的讨论、决定或继续任务：定位 `cwd` 后使用 `vault-recall`，把项目路径显式传给 Vault 的 `recent` / `search` / `show`，按需读取相关会话。
- Resume 与 Vault 的来源 ID 存在差异（例如 Resume 的 `codex` 对应 Vault 的 `openai-codex`）；优先从 Vault 查询结果取得它的 `agent` 和 `sessionId`，不要把 Resume 来源名直接当成 Vault 参数。
- 需要展示 Resume 会话引用时，原样使用结果中的 `AGENT` 和完整 `SESSION_ID`，格式为 `vani:<agent>:<session-id>`。引用单独占一行，不放进代码块。

## 范围与操作边界

- 项目列表来自受支持 Agent 历史会话的 `cwd` 聚合，覆盖有会话记录的目录；未命中表示索引里没有匹配项。新建但尚未产生会话的项目需要从用户提供的位置查找。
- `projects` / `sessions` 读取现有索引，不自动刷新。索引可包含旧路径；最近活跃和会话标题用于定位，项目当前进度需要读代码或相关会话确认。
- 默认只查询本机已有索引。扫描本地来源用 `resume scan`；访问 Amp 云端需要用户授权，再显式使用 `resume scan --with-cloud`。向 Vault 读取云端正文也遵循 `vault-recall` 的授权规则。
- 自动化查询使用明确子命令，不运行裸 `resume` 或 `resume tui`，它们会打开交互式选择器。
- 查询项目不运行 `resume init`，不创建分组或白板，不调用白板 `--draft --execute`。`resume board` 会在缺失时创建文件，不能作为纯读取接口。
- 用户明确要求恢复或启动会话时，先核对 `resume help` 中的命令；动作默认 dry-run，实际执行需要相应授权。不要仅因找到项目就启动另一个 Agent。
