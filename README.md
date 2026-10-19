# skills

Claude Code 自定义技能集合仓库。

## 包含的技能

- [generate-changelog](generate-changelog/README.md) — 自动生成结构化的 Git 变更日志
- [claude-md-management](claude-md-management/README.md) — 管理 CLAUDE.md 指令文件和 .claude/rules/ 规则文件
- [code-explain](code-explain/README.md) — 代码解读专家，帮助理解代码意图、流程与关键设计

## 安装

将所需技能目录复制到 `~/.claude/skills/` 下即可：

```bash
# 安装全部技能
cp -r */ ~/.claude/skills/

# 或只安装单个技能
cp -r generate-changelog ~/.claude/skills/
cp -r claude-md-management ~/.claude/skills/
```
