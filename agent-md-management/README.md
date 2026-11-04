# agent-md-management

创建、更新、审计或重组通用编码 agent 的 `AGENTS.md` 指令文件与下级目录中的嵌套 `AGENTS.md`，帮助 agent 按需加载仓库上下文。

支持的功能：
- 初始化或刷新根级 `AGENTS.md`
- 管理下级目录中的嵌套 `AGENTS.md`（就近优先、按需加载）
- 管理 monorepo 或子系统的本地指令
- 通过引用导入外部文件（README、`docs/`、`CONTRIBUTING.md` 等）
- 校验指令文件、引用链接和作用域一致性

## 安装

```bash
cp -r agent-md-management ~/.claude/skills/
```

## 使用

安装后，直接用自然语言触发：

- `为这个仓库创建 AGENTS.md`
- `刷新 AGENTS.md 和下级目录中的指令文件`
- `审计这个仓库的 AGENTS.md 是否过时`
- `把很长的 AGENTS.md 拆成嵌套 AGENTS.md`
- `为 monorepo 的 backend/ 和 frontend/ 添加各自的 agent 指令`
- `检查 AGENTS.md 里的引用是否都有效`
- `减少 agent 启动时加载的文档上下文`
