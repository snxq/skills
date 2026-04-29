---
name: generate-changelog
description: 在最近的两个 tag 之间生成结构化 changelog。当用户要求生成 changelog、release notes、版本更新说明、查看两个版本之间的改动、询问某个版本包含哪些变更时，都应使用此 skill。即使用户没有明确说"changelog"，只要涉及版本变更总结的场景都应触发。当用户指定了具体的 tag 或版本号时，生成对应范围的 changelog。
---

# 生成 Changelog

分析两个 tag 之间的所有提交，生成结构化的变更日志。默认使用 origin 最近的两个 tag，也支持用户指定 tag。

## 工作流程

### 第 1 步：刷新并识别 tag

先尽量刷新远程 tag，但不要硬依赖 `origin`：

```bash
git remote get-url origin >/dev/null 2>&1 && git fetch --tags origin || true
git tag --sort=-creatordate | head -2
```

如果仓库没有 `origin`、远程名不同或 fetch 失败，继续使用本地 tag，并在最终输出中说明远程 tag 未刷新。不要因为没有 `origin` 就终止 changelog 生成。

### 第 2 步：确定 tag 范围

根据用户输入确定 `OLD_TAG` 和 `NEW_TAG`：

- 未指定 tag：使用 `git tag --sort=-creatordate | head -2`，第一行是 `NEW_TAG`，第二行是 `OLD_TAG`。
- 指定单个 tag（如"生成 v1.4.3 的 changelog"）：该 tag 为 `NEW_TAG`，按 `--sort=-creatordate` 找到时间上紧邻的更早版本作为 `OLD_TAG`。
- 指定两个 tag：支持 `OLD_TAG..NEW_TAG`、`OLD_TAG 到 NEW_TAG`、`从 OLD_TAG 到 NEW_TAG`、`比较 OLD_TAG 和 NEW_TAG` 等表达。把较旧/基准版本作为 `OLD_TAG`，较新/目标版本作为 `NEW_TAG`。
- 如果两个 tag 的顺序与表达不一致或无法判断，按 tag 创建时间或提交拓扑推断；仍然模糊时询问用户确认，不要猜测。

对用户指定的每个 tag，先验证它存在：

```bash
git rev-parse --verify --quiet TAG_NAME^{tag} || git rev-parse --verify --quiet TAG_NAME^{commit}
```

如果任一指定 tag 不存在，停止并告诉用户缺失的 tag。

### 第 3 步：收集两个 tag 之间的所有提交

```bash
git log OLD_TAG..NEW_TAG --format="%H %s" --no-merges
```

`--no-merges` 过滤掉真正的 merge commit（多 parent）。但项目中 squash merge 的 PR 合入提交标题通常匹配 `^Merge #\d+ into \S+`（例如合入 `main`、`master`、`develop` 或 release 分支），它们只有一个 parent，`--no-merges` 无法过滤。这类提交需要保留并从 body 中提取实际变更描述。

如果过滤后没有任何提交，输出"两个 tag 之间没有实质性变更"并结束。

### 第 4 步：必要时阅读提交详情

如果某条提交的一行描述不够清晰，无法准确分类，需要查看该提交的具体改动来理解实际变更。

首先获取提交哈希（第 2 步已获取）。然后根据提交类型分别处理：

#### 普通提交（标题不以 "Merge #" 开头）

直接使用标题作为变更描述。如果描述不够清晰，逐层深入了解：

1. **查看改动了哪些文件**（大多数情况足够判断分类）：
   ```bash
   git show --stat COMMIT_HASH
   ```

2. **查看完整的 diff 内容**（当提交描述模糊、无法仅从文件名判断分类时，例如"fix: 修复问题"、"chore: 更新"等无具体描述的提交）：
   ```bash
   git show COMMIT_HASH
   ```
   通过阅读实际代码变更来确定该提交的性质（新功能 / bug 修复 / 其他），从而准确分类。

#### Squash Merge PR 提交（标题匹配 `^Merge #\d+ into \S+`）

这类提交的实际变更描述在 body 中，需要从 body 提取：

```bash
git log COMMIT_HASH -1 --format="%B"
```

body 中通常包含：
- 第一行：实际变更描述（如"工单增加资源类型"、"修复已规划的冷热标签"）
- 后续行：分支信息、签名等（忽略）

提取 body 第一行有意义的描述作为该提交的变更内容。如果 body 描述仍然模糊，再用 `git show --stat` 或 `git show` 深入了解。

### 第 5 步：分类并生成 changelog

根据提交消息前缀和变更性质，将每条提交归入以下三个类别之一：

- **Feature**：以 `feat` 开头的提交，或添加了新功能、新 API、新页面、新能力的变更
- **Bug fix**：以 `fix` 开头的提交，或修复缺陷、纠正异常行为、解决回归问题的变更
- **Other**：其他所有变更 — 重构、文档、配置变更、测试、性能优化、依赖更新、杂项

### 第 6 步：输出格式

使用以下模板。语言与提交消息保持一致（提交是中文则输出中文，英文则输出英文）。

```markdown
# Changelog

OLD_TAG → NEW_TAG

## Feature
- **scope**: 描述内容
- **scope**: 描述内容

## Bug fix
- **scope**: 描述内容

## Other
- 描述内容
```

格式规则：
- 提取 scope（feat/fix 后括号内的部分），以加粗显示
- 清理描述内容 — 去掉 conventional commit 前缀（`feat(scope):`、`fix:` 等），只保留人类可读的描述
- 如果提交没有 scope，使用 `- 描述内容` 格式（不加粗任何内容），与有 scope 的条目保持对齐
- 合并相同 scope（仅在同一分类内合并）：如果同一个分类下同一个 scope 有多条提交，使用多级无序列表展示，例如：
  ```markdown
  - **demand**: 新增离线设备清单 API
    - 优化需求扣减逻辑
    - 添加离线子需求带宽汇总字段
  ```
- 去重：如果多条提交描述的是同一项变更，保留最完整的一条
- 过滤无意义条目：如果某条提交的描述过于模糊（如"fix"、"fix test"、"chore"等），且无法通过查看 diff 补充出有意义的描述，则跳过该条目，不纳入 changelog
- 每个分类内按重要性排序（主要功能在前，小修复在后）
- 如果某个分类没有条目，省略该分类（不要显示空的 "Bug fix" 标题）
- 不要编造或推测提交中不存在的变更
