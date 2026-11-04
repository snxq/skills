# skills

通用 Agent 自定义技能集合仓库。

## 包含的技能

- [generate-changelog](generate-changelog/README.md) — 自动生成结构化的 Git 变更日志
- [agent-md-management](agent-md-management/README.md) — 管理 AGENTS.md 指令文件与下级目录中的嵌套 AGENTS.md
- [code-explain](code-explain/README.md) — 代码解读专家，帮助理解代码意图、流程与关键设计
- [create-git-worktree](create-git-worktree/README.md) — 在仓库 .worktrees 下创建 worktree，按功能描述自动起名

## 安装

将所需技能目录复制到所用 Agent 的 skills 目录（如 Claude Code 的 `~/.claude/skills/`）下即可：

```bash
# 安装全部技能
cp -r */ ~/.claude/skills/

# 或只安装单个技能
cp -r generate-changelog ~/.claude/skills/
cp -r agent-md-management ~/.claude/skills/
```
