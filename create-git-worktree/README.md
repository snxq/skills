# create-git-worktree

在项目仓库内的 `.worktrees/` 下创建 linked worktree，用于并行开发而不动当前工作区。

支持的功能：
- 有功能描述时自动翻译成英文 kebab-case 名称，并从主仓库当前 HEAD 新建同名分支
- 无描述时询问用户输入名称
- 名称对应分支已存在时直接检出，不重复新建；本地没有但远端有同名分支时以远端分支为起点新建并跟踪
- 自动追加或新建 `.gitignore` 忽略 `.worktrees/`，避免父仓库 status 噪音
- 冲突（路径占用、分支已被其他 worktree 检出）时停下提示，不强行覆盖

## 安装

```bash
cp -r create-git-worktree ~/.claude/skills/
```

## 使用

安装后，直接用自然语言触发：

- `创建一个 worktree 来修复登录超时`
- `新建一个 worktree，分支名 release-1.2`
- `移除 .worktrees/fix-login-timeout`
