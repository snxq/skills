# skills

Claude Code 自定义技能集合仓库。

## 包含的技能

### generate-changelog

分析两个 Git tag 之间的所有提交，自动生成结构化的变更日志（Changelog）。

支持的功能：
- 自动识别最近的两个 tag
- 支持指定 tag 范围
- 提交自动分类为 Feature / Bug fix / Other
- 兼容 Conventional Commits 规范和 Squash Merge PR
- 模糊提交会深入查看 diff 补充描述

### agents-md-management

创建、更新、审计或重组面向 coding agent 的 `AGENTS.md` 指令文件，帮助 agent 按需加载仓库上下文。

支持的功能：
- 初始化或刷新根级 `AGENTS.md`
- 管理 monorepo 或嵌套目录的 `AGENTS.md`
- 将详细说明下沉到被索引的支持文件
- 校验索引文件、链接和指令作用域
- 遵循 agents.md 指南减少上下文膨胀

## 安装

将所需技能目录复制到 `~/.claude/skills/` 下即可：

```bash
# 安装全部技能
cp -r */ ~/.claude/skills/

# 或只安装单个技能
cp -r generate-changelog ~/.claude/skills/
cp -r agents-md-management ~/.claude/skills/
```

## 使用

安装后，在 Claude Code 中直接用自然语言触发对应的技能：

- `生成 changelog`
- `生成 v1.4.3 的 changelog`
- `查看最近两个版本的改动`
- `生成 release notes`
- `为这个仓库创建 AGENTS.md`
- `刷新已有 AGENTS.md 并同步索引文件`
