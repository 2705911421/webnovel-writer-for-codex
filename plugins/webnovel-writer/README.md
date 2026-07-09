# Webnovel Writer Codex Plugin Package

这是 `webnovel-writer` 的 Codex 插件包目录。

完整安装与使用说明见仓库根目录 [README.md](../../README.md)。

## 包内组件

| 类型 | 位置 |
| --- | --- |
| Codex 插件清单 | `.codex-plugin/plugin.json` |
| Codex 适配层 | `skills/webnovel-codex/SKILL.md` |
| 创作工作流 Skills | `skills/webnovel-*/SKILL.md` |
| 子代理提示词 | `agents/*.md` |
| Python 工具入口 | `scripts/webnovel.py` |
| 写作资料与模板 | `references/`, `templates/` |
| 只读 Dashboard | `dashboard/` |

## 快速检查

```powershell
python -X utf8 .\scripts\webnovel.py --help
```

## 上游

原始 Claude Code 插件来自 [lingfengQAQ/webnovel-writer](https://github.com/lingfengQAQ/webnovel-writer)。

本目录仅加入 Codex 适配清单与兼容说明，主体能力、脚本、资料库和协议仍继承上游项目。
