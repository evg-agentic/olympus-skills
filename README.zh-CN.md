# Olympus Skills

[Olympus](https://github.com/evg-agentic) agentic runtime 的 agent skill 集合 ——
operator 白皮书 + 通用知识 skill。兼容任何读 Markdown skill 的 agent(Claude Code、Cursor、Codex …)。

**[English](./README.md)**

## Skills

| Skill | 作用 |
|---|---|
| [`olympus-operator`](./skills/olympus-operator/SKILL.md) | 通过 `@evg-olympus/cli` 操作整个 Olympus 系统 —— 模块联动模型、18 条隐性依赖检查表、5 条端到端维护 playbook。旗舰 skill。 |

通用知识 skill(提示词工程、叙事、影视制作 …)逐步增量加入 —— 每个都是自包含 Markdown,不依赖 Olympus runtime。

## 安装

任选一种,都是把这个 repo 克隆进 agent 的 skills 目录:

```bash
npx skills add evg-agentic/olympus-skills      # 推荐,cross-agent
gh skill install evg-agentic/olympus-skills     # GitHub CLI 2.90+
```

或在 Claude Code 里:

```
/plugin marketplace add evg-agentic/olympus-skills
/plugin install olympus@olympus-operator
```

## 使用 `olympus-operator`

白皮书驱动 Olympus CLI,走带门禁的 Admin API:

```bash
npm install -g @evg-olympus/cli
olympus login        # 浏览器 device-flow;必须在 operator allowlist 里
```

然后让 agent 去查看、组装或修改 Olympus 系统 —— 它会遵循 skill 里的联动模型和隐性依赖检查表。runnable bundle(沙箱脚本)**不**发布到这里;它们在 Olympus 平台内,通过 CLI 管理。

## 仓库结构

```
skills/<name>/SKILL.md     每个 skill 一个目录(agent 加载这个文件)
```

## License

MIT —— 见 [LICENSE](./LICENSE)。
