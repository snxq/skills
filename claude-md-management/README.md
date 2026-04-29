# claude-md-management

创建、更新、审计或重组面向 Claude Code 的 `CLAUDE.md` 指令文件和 `.claude/rules/` 规则文件，帮助 agent 按需加载仓库上下文。

支持的功能：
- 初始化或刷新根级 `CLAUDE.md`
- 管理 `.claude/rules/` 规则文件（支持 `paths` 条件加载）
- 管理 monorepo 或嵌套目录的 `CLAUDE.md`
- 通过 `@` 导入引用外部文件（如 `AGENTS.md`、README）
- 校验指令文件、链接和作用域一致性

## 安装

```bash
cp -r claude-md-management ~/.claude/skills/
```

## 使用

安装后，在 Claude Code 中直接用自然语言触发：

- `为这个仓库创建 CLAUDE.md`
- `刷新 CLAUDE.md 和 .claude/rules/ 规则文件`
- `审计这个仓库的 CLAUDE.md 是否过时`
- `把很长的 CLAUDE.md 拆成 .claude/rules/ 规则文件`
- `为 monorepo 的 backend/ 和 frontend/ 添加各自的 agent 指令`
- `检查 CLAUDE.md 里的 @ 导入是否都有效`
- `减少 agent 启动时加载的文档上下文`
