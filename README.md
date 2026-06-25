# my-awesome-skills

一组面向 [Agent Skills 标准](https://developers.openai.com/codex/skills)（`SKILL.md`）的自用 skill，
可在 **Claude Code** 和 **OpenAI Codex** 上加载。仓库按 `<skill>/<platform>/` 分平台组织，
每个平台一份针对其能力裁剪过的实现；Claude 侧同时是一个 plugin marketplace，可用 `/plugin` 一键安装
（见[部署](#部署)）。

## Skills

### `verify-claims` — 关键断言验证

对输出中的关键断言（版本号、API 行为、因果推断、可直接执行的技术建议）做验证，并在断言后做
内联标注（`[✅已验证: ...]` / `[⚠️部分验证: ...]` / `[❗未验证: ...]`）。优先查本地源
（代码、git、锁文件、配置），不足时再联网查官方文档/规范/发布说明。专治易幻觉领域：兼容性矩阵、
具体参数值、版本依赖。

- **Claude 版**：联网检索按「默认 search 工具（`WebSearch`/`WebFetch`）→ 其他可用 websearch 工具
  → 都没有则给出特别告警并标注未验证」的顺序降级，不绑定特定工具。
- **Codex 版**：检索步骤为「使用 Codex 可用的联网搜索/浏览能力」，同样不绑定特定工具；
  另带 `agents/openai.yaml` 配置 Codex app 的 UI 元数据与隐式调用策略。

### `fork-context` — 上下文分叉

在一个很长的会话里碰到「相关但跑题」的想法时，把与该 tangent 相关的上下文萃取成一份自包含
brief 文件，并打印一段可直接粘进**全新 `claude` 会话**的 paste-prompt——既能在干净上下文里
探索分叉，又不污染当前主线会话。

实现的核心是：萃取重活全部在一个 `fork` 类型 subagent 里完成（继承当前完整对话、但在隔离
上下文运行），主会话只负责 dispatch 和打印结果，因此主会话上下文净增量极小。

> ⚠️ 该机制依赖 Claude Code 的 `fork` subagent，**Codex 无对应能力，故只提供 Claude 版**。

## 支持状态

| Skill | Claude Code | OpenAI Codex | 说明 |
|---|:---:|:---:|---|
| `verify-claims` | ✅ | ✅ | 两平台各一份实现；Codex 版附 `agents/openai.yaml` |
| `fork-context`  | ✅ | ❌ | 依赖 Claude Code 的 `fork` subagent，Codex 无对应机制 |

### 依赖

- `verify-claims` 不依赖任何外部 skill。联网检索优先用环境默认 search 工具
  （Claude Code 为 `WebSearch`/`WebFetch`），不可用时降级到其他 websearch 工具，
  完全没有联网工具时会给出特别告警并把相关断言标注为未验证。
- `fork-context` 无外部 skill 依赖，但要求运行环境支持 `fork` 类型 subagent（Claude Code）。

## 部署

- **Claude Code**：本仓库就是一个 plugin marketplace，直接用 `/plugin` 安装，无需手动拷贝。
- **OpenAI Codex**：Codex 不支持 Claude 的 plugin 机制，把对应 `codex/` 目录拷到 `~/.codex/skills/` 即可。

### Claude Code（plugin marketplace）

仓库根的 `.claude-plugin/marketplace.json` 把每个 skill 注册为一个 plugin。在 Claude Code 里：

```text
/plugin marketplace add jessehui/my-awesome-skills
/plugin install verify-claims@my-awesome-skills
/plugin install fork-context@my-awesome-skills
```

第一行注册本 marketplace（名字 `my-awesome-skills` 取自 `marketplace.json` 的 `name`，
与仓库名无关）；后两行各装一个 plugin，`@` 后面是 marketplace 名。装完即可用
`/verify-claims`、`/fork-context` 显式调用，或让 Claude 按 `description` 自动触发。

> 想本地迭代、先不推 GitHub，可用本地路径：`/plugin marketplace add /path/to/my-awesome-skills`。
> 更新：`/plugin marketplace update my-awesome-skills` 后重装对应 plugin。

### OpenAI Codex

个人级 skill 目录为 `$CODEX_HOME/skills/<skill-name>/`（默认 `~/.codex/skills/`；
项目级为仓库内 `.agents/skills/`）。先拉仓库，再把 `codex/` 整个目录内容
（含 `agents/openai.yaml`）复制到 `~/.codex/skills/verify-claims/`：

```bash
git clone https://github.com/jessehui/my-awesome-skills.git
cd my-awesome-skills
mkdir -p ~/.codex/skills/verify-claims
cp -r verify-claims/codex/. ~/.codex/skills/verify-claims/
```

> Codex 在已存在同名 skill 目录时会拒绝覆盖安装；重装请先删除旧目录。
> 修改后 Codex 一般会自动检测；若未生效，重启 Codex。

装好后在 Codex 里输入 `$verify-claims` 显式调用，或由 Codex 按 `description` 自动选择。
`fork-context` 不提供 Codex 版（见上）。

## 仓库结构

```
my-awesome-skills/
├── .claude-plugin/
│   └── marketplace.json              # marketplace 清单，列出两个 plugin
├── verify-claims/
│   ├── claude/                       # Claude plugin root（被 marketplace 引用）
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/verify-claims/SKILL.md
│   └── codex/                        # Codex 手动安装用
│       ├── SKILL.md
│       └── agents/openai.yaml
└── fork-context/
    └── claude/                       # 仅 Claude
        ├── .claude-plugin/plugin.json
        └── skills/fork-context/SKILL.md
```

每个 `<skill>/claude/` 目录本身就是一个 Claude plugin（含 `.claude-plugin/plugin.json`
和 `skills/<name>/SKILL.md`），由根目录的 `marketplace.json` 通过相对路径引用。
各 skill 目录下另有 `*-design.md` 设计文档，记录设计取舍，不进 plugin、部署时无需复制。

## 参考

- [Agent Skills — Codex / OpenAI Developers](https://developers.openai.com/codex/skills)
- [Claude Code — Skills](https://docs.claude.com/en/docs/claude-code/skills)
