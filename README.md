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
└── vault-recall/
    └── SKILL.md
```

后续新增 Skill 时继续使用：

```text
vani-skills/<skill-name>/SKILL.md
```

Skill 应按用户要完成的工作流划分，而不是照搬 Vani 内部的代码模块边界。

## 当前 Skills

### `vault-recall`

通过 `vault recent`、`vault search` 和 `vault show` 定位并读取过去的 Agent 对话。
使用前需要先安装 `vani-vault`，并确保 `vault` 命令位于 `PATH`。

## 安装

使用支持 Agent Skills 的安装器，从本目录选择需要的 Skill。例如在当前星座工作区中：

```bash
npx skills add ./vani-skills -s vault-recall -g -y
```

安装后由各 Agent 自己将 Skill 发现到对应的用户级目录。不要手工维护多份
`SKILL.md`；本目录是 Vani Skills 的统一来源。
