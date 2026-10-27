# Vani Skills

Vani 各模块面向 Agent 的统一 Skills 目录。

这里存放可跨 Claude Code、Codex、Amp 等兼容 Agent 使用的工作流说明。Vani 采用
**CLI-first + Skill-first**：CLI 提供稳定、可测试的机器能力，Skill 只负责告诉 Agent
何时调用、如何组合命令，以及怎样控制输出和处理失败。只要本地 CLI 足以完成集成，
就不额外构建 MCP Server。

## 目录结构

每个 Skill 使用一个独立目录，并以 `SKILL.md` 作为入口：

```text
vani-skills/
├── README.md
├── resume-discover/SKILL.md
├── vault-recall/SKILL.md
├── using-rws/SKILL.md
└── mcp/vani-tasks.json
```

后续新增 Skill 时继续使用：

```text
vani-skills/<skill-name>/SKILL.md
```

Skill 应按用户要完成的工作流划分，而不是照搬 Vani 内部的代码模块边界。

## 当前 Skills

### `resume-discover`

通过 `resume projects`、`resume sessions` 和项目分组定位工作区，再衔接 Vault 查询历史正文。
需要本机 `resume` CLI；扫描和云端读取遵循 skill 中的授权边界。

### `vault-recall`

通过 `vault recent`、`vault search` 和 `vault show` 定位并读取过去的 Agent 对话。
支持多 Agent 来源，正文默认使用 `--format conversation-json`。需要支持该格式的新版
`vault` CLI；旧版本应先升级，不将 Markdown 视为完整事件图。

### `using-rws`

用本地 `rws` CLI 将 Git 工作区快照和命令发送到用户明确指定的 SSH Linux target，
查询、取消和拉取持久任务。需先安装 `rws` 并配置 SSH；`gpu-box` 是示例 SSH 别名。

## 安装

Vani 的 Settings → Connectors 为这三个运行集成预填公开 Git URL
`https://github.com/Nichengjin/vani-skills.git`、`master` 和对应的根级 skill 目录。
审阅时解析到具体 commit 并形成不可变快照，确认后记录来源；更新仍需重新审阅。
其他支持 Agent Skills 的安装器也可直接从公开仓库选择，例如：

```bash
npx skills add Nichengjin/vani-skills -s vault-recall -g -y
```

安装后由各 Agent 自己将 Skill 发现到对应的用户级目录。不要手工维护多份
`SKILL.md`；本目录是 Vani Skills 的统一来源。

## MCP 配置模板

`mcp/vani-tasks.json` 是数据模板，不是可执行脚本。Vani 在用户配置缺失的 Tasks MCP 时
从公开仓库读取，校验 schemaVersion 1、`vani-tasks`／stdio／`tasks mcp` 契约，
将 `tasks` 替换为本机探测到的 CLI 路径，再进入已有原生配置预览／确认流程。
模板不包含环境变量、密钥、机器路径或安装命令；下载失败不回退私有源码仓库。
已有注册仅进入本机管理，不用模板覆盖用户配置。`vani_delivery` 仍由运行层按会话注入。

本仓库分发 Agent 指令与模板，CLI 二进制由公开的
[vani-releases](https://github.com/Nichengjin/vani-releases) 分发流程负责；这里不承诺 CLI 频道已发布。
