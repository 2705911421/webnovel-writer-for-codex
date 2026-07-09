# Webnovel Writer For Codex

这是 [lingfengQAQ/webnovel-writer](https://github.com/lingfengQAQ/webnovel-writer) 的 Codex 适配版。

原项目是面向 Claude Code 的长篇中文网文创作插件，提供新书初始化、卷纲规划、章节写作、质量审查、状态查询、长期记忆、RAG 检索和只读 Dashboard。本仓库的目标不是重写它，而是把原有能力迁移成 Codex 可以安装和识别的本地插件。

## 相较原版的改动

| 项目 | Claude Code 原版 | Codex 适配版 |
| --- | --- | --- |
| 插件清单 | `.claude-plugin/plugin.json` | 新增 `.codex-plugin/plugin.json` |
| 分发入口 | Claude Code Marketplace | 新增 Codex marketplace：`.agents/plugins/marketplace.json` |
| 触发方式 | `/webnovel-*` slash command | 保留 `/webnovel-*` 作为自然语言意图，也支持中文描述 |
| 环境变量 | 依赖 `CLAUDE_PLUGIN_ROOT`、`CLAUDE_PROJECT_DIR` | 新增 `webnovel-codex` 适配层，映射到 Codex 工作区和插件根目录 |
| 交互工具 | `AskUserQuestion`、`Agent tool`、`WebSearch/WebFetch` | 在 Codex 中改为聊天确认、可用子任务/子代理、按需联网检索 |
| Skill 格式 | 含 Claude 专属字段，如 `argument-hint` | 已清理为 Codex 可校验格式 |
| 主体能力 | skills、agents、scripts、references、templates、dashboard | 原样保留，并补充 Codex 使用说明 |

保留的原版能力包括：

- `webnovel-init`：深度初始化新书项目。
- `webnovel-plan`：生成卷纲、时间线和章纲。
- `webnovel-write`：按上下文、审查、润色、提交链路写章节。
- `webnovel-review`：多维度审查章节并生成报告。
- `webnovel-query`：只读查询角色、伏笔、设定和运行时信息。
- `webnovel-learn`：把有效写法沉淀到项目长期记忆。
- `webnovel-dashboard`：启动只读可视化面板。
- `webnovel-doctor`：检查项目结构、依赖、RAG 和数据链健康度。

## 当前仓库结构

```text
.
├── .agents/plugins/marketplace.json          # Codex marketplace
└── plugins/webnovel-writer/
    ├── .codex-plugin/plugin.json             # Codex 插件清单
    ├── .claude-plugin/plugin.json            # 保留上游 Claude 清单，便于同步
    ├── skills/webnovel-codex/SKILL.md        # Codex 兼容层
    ├── skills/webnovel-*/SKILL.md            # 原创作工作流
    ├── agents/                               # 原子代理提示词
    ├── scripts/                              # Python 运行时工具
    ├── references/                           # 题材、技法、审查资料
    ├── templates/                            # 输出模板和题材模板
    └── dashboard/                            # 只读 Dashboard
```

## 安装到 Codex

先克隆仓库：

```powershell
git clone https://github.com/2705911421/webnovel-writer-for-codex.git
cd webnovel-writer-for-codex
```

注册本仓库的 Codex marketplace：

```powershell
codex plugin marketplace add .agents/plugins
```

安装插件：

```powershell
codex plugin add webnovel-writer@webnovel-writer-for-codex
```

如果你使用的是 Codex 桌面版，也可以在插件界面中选择 `Webnovel Writer` 安装。安装后建议新开一个 Codex 线程，让新 skill 被完整加载。

## 安装 Python 依赖

插件脚本需要 Python 3.10+：

```powershell
python -m pip install -r plugins/webnovel-writer/scripts/requirements.txt
```

Dashboard 运行时还可能需要：

```powershell
python -m pip install -r plugins/webnovel-writer/dashboard/requirements.txt
```

如果要使用语义 RAG，请在书项目根目录配置 `.env`。未配置 Embedding/Rerank 时，系统会自动回退到 BM25 关键词检索。

最小 `.env` 示例：

```env
EMBED_BASE_URL=https://api-inference.modelscope.cn/v1
EMBED_MODEL=Qwen/Qwen3-Embedding-8B
EMBED_API_KEY=your_embed_api_key

RERANK_BASE_URL=https://api.jina.ai/v1
RERANK_MODEL=jina-reranker-v3
RERANK_API_KEY=your_rerank_api_key
```

## 在 Codex 中怎么用

安装后，可以直接用原来的命令形式：

```text
/webnovel-init
/webnovel-plan 1
/webnovel-write 1
/webnovel-review 1-5
/webnovel-query 伏笔
/webnovel-learn "这个开篇钩子有效"
/webnovel-dashboard
/webnovel-doctor
```

也可以用自然语言：

```text
用 Webnovel Writer 初始化一本新网文。
用 Webnovel Writer 规划第 1 卷。
用 Webnovel Writer 写第 12 章。
用 Webnovel Writer 审查第 1 到第 5 章。
用 Webnovel Writer 查一下主角当前状态和未回收伏笔。
启动 Webnovel Writer 的 Dashboard。
```

Codex 会先读取 `skills/webnovel-codex/SKILL.md` 的兼容说明，再进入对应的具体工作流。

## 典型工作流

### 1. 初始化一本新书

在一个适合作为写作工作区的目录里打开 Codex，然后输入：

```text
/webnovel-init
```

Codex 会分阶段收集题材、主角、金手指、世界观、创意约束等信息。确认后会生成书项目目录，包含 `.webnovel/`、`.story-system/`、`设定集/`、`大纲/`、`正文/` 等文件夹。

### 2. 规划第 1 卷

```text
/webnovel-plan 1
```

该流程会基于总纲生成卷纲、时间线、章纲，并把新增设定增量写回设定集。

### 3. 写章节

```text
/webnovel-write 1
```

写章流程会读取上下文、生成正文、调用审查、润色、提交事实、更新状态，并尽量保留可恢复的中间产物。

### 4. 审查章节

```text
/webnovel-review 1-5
```

审查结果会写入审查报告，并把质量指标同步到项目状态中。

### 5. 查询项目状态

```text
/webnovel-query 主角当前目标和未回收伏笔
```

查询流程是只读的，适合在规划和写作前确认设定、角色、伏笔、时间线和运行时信息。

### 6. 打开 Dashboard

```text
/webnovel-dashboard
```

Dashboard 是只读面板，用来查看项目状态、章节内容、角色关系、伏笔和追读力数据。

### 7. 体检项目

```text
/webnovel-doctor --deep
```

当写作链路卡住、依赖缺失、RAG 配置异常或状态文件不一致时，优先运行 doctor。

## 命令行验证

在仓库根目录运行：

```powershell
python -X utf8 .\plugins\webnovel-writer\scripts\webnovel.py --help
```

如果已经有一个书项目，可以运行只读体检：

```powershell
python -X utf8 .\plugins\webnovel-writer\scripts\webnovel.py --project-root "<你的书项目路径>" doctor --format text
```

## 当前限制

- 这是 Codex 适配版，不是上游官方发布。
- 原项目中的 hooks 仍保留在包内，但 Codex 当前不会按 Claude Code 的 hook 机制自动执行它们。
- 原提示词中仍会出现少量 Claude 术语；实际执行时由 `webnovel-codex` 适配层解释为 Codex 行为。
- 如果直接调用 CLI，需要自己传入正确的 `--project-root`。
- Windows 用户建议使用 `python -X utf8`，避免中文路径和输出编码问题。

## 如何同步上游

本仓库保留了上游插件主体。后续同步新版时，推荐流程是：

1. 从 [lingfengQAQ/webnovel-writer](https://github.com/lingfengQAQ/webnovel-writer) 获取新版 `webnovel-writer/` 插件包。
2. 覆盖本仓库的 `plugins/webnovel-writer/` 主体文件。
3. 保留或重新应用：
   - `.codex-plugin/plugin.json`
   - `skills/webnovel-codex/SKILL.md`
   - 各 `skills/webnovel-*/SKILL.md` 顶部的 Codex migration note
   - Codex 不兼容 frontmatter 清理，如 `argument-hint`、多余 `version`
4. 运行插件校验和 CLI smoke test。

## 上游与协议

- 上游项目：[lingfengQAQ/webnovel-writer](https://github.com/lingfengQAQ/webnovel-writer)
- Codex 适配仓库：[2705911421/webnovel-writer-for-codex](https://github.com/2705911421/webnovel-writer-for-codex)
- 协议：GPL-3.0，详见 [LICENSE](plugins/webnovel-writer/LICENSE)

使用、修改和再分发时请遵守 GPL-3.0，并保留原作者署名。
