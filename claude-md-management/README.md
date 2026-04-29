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
