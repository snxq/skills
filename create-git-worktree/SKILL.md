---
name: create-git-worktree
description: 在项目仓库的 .worktrees 下创建或移除 git worktree：有功能描述时按描述自动起名并新建分支，否则询问用户输入名称
tags: ["git", "worktree", "branch"]
allowed-tools: ["Bash", "Write", "Edit", "AskUserQuestion"]
---

# 创建 Git Worktree

在项目仓库内的 `.worktrees/` 下创建 linked worktree，用于并行开发而不动当前工作区。git 2.56 实测支持嵌套（worktree 建在仓库工作区内），父仓库会把 `.worktrees/` 整体视为未跟踪文件，需用 `.gitignore` 忽略。

## 流程

1. 定位主仓库根：
   - `G=$(git rev-parse --path-format=absolute --git-common-dir)`，候选根 = `dirname "$G"`。
   - 验证候选根属于本仓库：`git -C <候选根> rev-parse --path-format=absolute --git-common-dir` 输出等于 `$G`。
   - 验证不通过（`.git` 为文件指针的布局：`--separate-git-dir`、远程 submodule）：主根无法可靠自动定位，不要猜测——用 `AskUserQuestion` 让用户确认主工作区路径；用户正在主工作区时，`git rev-parse --show-toplevel` 即正确答案，可作默认选项。
   - 不要单独用 `--show-toplevel`：在 linked worktree 内它返回该 worktree 自己的根。
   当前目录不在 git 仓库内时，先提示用户进入 git 项目目录再继续。`.worktrees/` 一律放在主仓库根下，后续命令均在主仓库根执行（`git -C "<主仓库根>"` 或先 cd 过去；路径含空格时引号不能省）。
2. 确定名称，三选一：
   - 用户已给出分支名/名称 → 若该名称是已有本地分支（`git show-ref --verify --quiet refs/heads/<名称>` 退出码 0，精确匹配、不做 glob）或同名远端分支（`git branch -r --list '*/<名称>'` 有输出，忽略含 `->` 的 symref 行如 `origin/HEAD`；名称含 `*`、`?`、`[` 等 glob 字符时跳过本项远端检查，按新名称处理）则原样采用，不做清洗（分支名大小写敏感）；否则按第 3 步清洗。
   - 只给出功能描述（如“修复登录超时”）→ 翻译成英文并转为 kebab-case（`fix-login-timeout`），无需再确认，直接采用。
   - 两者都没有 → 用 `AskUserQuestion` 或直接提问让用户输入名称。
3. 清洗名称（用户给出且为已有分支的名称跳过本步）：小写；中文转英文；空格、下划线转 `-`；去掉 `[a-z0-9./-]` 以外字符；每个 `/` 分隔的段去掉首尾的 `.`、段尾的 `.lock`（反复执行直到不再变化）；然后折叠连续的 `-`、`/`、`.` 为单个，去掉空段；整名不以 `-` 或 `/` 开头结尾；尽量简短（≤ 30 字符）。清洗结果为空，或 `git check-ref-format refs/heads/<name>` 校验不通过（退出码非 0）→ 回到第 2 步询问名称。
4. 冲突检查：
   - 先 `git worktree prune` 清理注册残留（目录已被外部删除的条目）；`locked` 状态的残留 prune 清不掉——`git worktree list --porcelain` 里带 `locked` 且目录不存在的，经用户确认后先 `git worktree unlock <路径>` 再 prune。
   - `.worktrees` 本身已存在且是文件 → 停下，提示用户处理。
   - `.worktrees/<name>` 路径已存在且非空 → 停下，提示换名或先 `git worktree remove .worktrees/<name>`；为空（如移除后残留的空目录）→ 继续。
   - 目标分支（`<name>`，或用户指定检出的既有分支）已检出 → 停下，提示换名或改用其他分支；同一分支不能同时检出到多个 worktree。判断方法：`git worktree list --porcelain`，解析 `branch refs/heads/<分支>` 行（主工作区也在其中；detached 状态没有 branch 行），去掉 `refs/heads/` 后与目标分支比较——不要直接 grep 输出文本，路径可能包含分支名字符串。
5. 确定分支与创建方式：
   - 名称对应的本地分支已存在（`git show-ref --verify --quiet refs/heads/<name>` 退出码 0）→ `git worktree add .worktrees/<name> <name>`（直接检出，不新建分支）。
   - 本地不存在、但有同名远端分支（`git branch -r --list '*/<name>'` 有输出）→ `git worktree add .worktrees/<name> -b <name> <该远端分支>`（以远端分支为起点并自动建立跟踪，而不是从主仓库 HEAD 新建；多个远端同名时优先 origin，其余情况询问用户；refs/remotes 是上次 fetch 的快照、可能过期，拿不准时先 `git fetch` 再检查）。
   - 都没有 → `git worktree add .worktrees/<name> -b <name>`（起点是执行命令处的当前 HEAD；第 1 步已保证在主仓库根执行，所以是主仓库 HEAD——若误在某个 worktree 内直接执行，起点会变成该 worktree 的 HEAD）。
   - 用户明确指定既有分支 X、名称仅作目录名 → `git worktree add .worktrees/<name> X`（X 为 `origin/…`、tag 或裸 SHA 时都会得到 detached worktree，需要本地分支时走上一项）。
   - git 会自动创建 `.worktrees` 父目录，无需手动 mkdir。
   - 创建失败时看错误：ref 锁定冲突（`cannot lock ref`，名称的父级段撞上已有分支，如 `feat/bar` 撞 `feat`）→ 换名重试。
6. `.gitignore`：`git check-ignore -q .worktrees/` 退出码为 1（未被忽略）时处理，避免父工作区 status 噪音和误提交：
   - 主仓库根已有 `.gitignore` → 追加一行 `.worktrees/`（文件不以换行结尾时先补换行，避免与上一行粘连）。
   - 没有 `.gitignore` → 新建 `.gitignore`，写入一行 `.worktrees/`。
   写完后用 `git check-ignore -q .worktrees/` 复核，退出码 0 才算完成。
7. 验证并报告：`git -C .worktrees/<name> status -sb` 确认分支正确，然后向用户报告 worktree 路径。

## 移除

- `git worktree remove .worktrees/<name>`；工作区有未提交改动或未跟踪文件时 git 会拒绝（exit 128），需先确认或加 `--force`。`--force` 会永久删除该 worktree 里的未提交改动和未跟踪文件，加之前先确认。
- 外部删除过的注册残留用 `git worktree prune` 清理（`locked` 条目需先 `git worktree unlock`）。

## 注意

- 新分支基于执行命令处的当前 HEAD（本流程即主仓库 HEAD）；任何工作区（含你所在的 worktree）未提交的改动都不会带入新 worktree。
- `git worktree add` 中途失败可能已创建分支但未建目录，重试前先确认该分支是否已存在。
- 仓库还没有首个提交（unborn HEAD）时，`-b` 会创建孤儿分支，而不是基于 HEAD。
- 主仓库处于 rebase/bisect 冲突中（HEAD detached）时，`-b` 会基于冲突中的中间提交而非分支尖端；创建前先确认主仓库状态。
- 主仓库启用 sparse checkout 时，新 worktree 继承稀疏配置（`core.sparseCheckout` 存于共享配置），文件集可能与预期不同。
- worktree 是嵌套在主工作区内的目录，父仓库不要在根目录跑 `git clean -x`（会把 `.worktrees/` 一起删掉）。
- worktree 内首次 `git push` 可能需要 `--set-upstream`。
- 仓库含 submodule 时，新 worktree 不会初始化 submodule（目录为空），需要时在 worktree 内运行 `git submodule update --init`。
- 创建嵌套 worktree 时偶发 `worktree clone fell back to plain checkout` 警告（路径重复的怪异长串），是 clone 优化的正常回退，不影响结果。
