---
name: claude-md-management
description: 当用户要求创建、编写、更新、刷新、审计或重组 CLAUDE.md 文件时使用；当需要管理 `.claude/rules/` 规则文件、monorepo 或嵌套 CLAUDE.md 指令、减少 agent 文档上下文膨胀、对编码 agent 指令应用渐进披露时使用。
---

# CLAUDE.md 管理

创建或更新面向 Claude Code 的仓库说明，让未来的 agent 获得正确上下文，而不是一次性加载整个项目的全部细节。

## Claude Code 的加载机制

理解 Claude Code 如何加载指令文件是管理它们的前提：

- **根 `CLAUDE.md`**：项目根目录（或 `.claude/CLAUDE.md`）在每次会话启动时自动加载，无论长度都完整读取。向上遍历父目录也会加载沿途的 `CLAUDE.md`。
- **嵌套 `CLAUDE.md`**：子目录中的 `CLAUDE.md` 不在启动时加载，而是在 Claude 读取该目录中的文件时按需加载。
- **`.claude/rules/*.md`**：放在 `.claude/rules/` 目录中的规则文件会被自动发现。无 `paths` frontmatter 的规则在启动时加载；带 `paths` 的规则在 Claude 处理匹配文件时才加载。
- **`@` 导入**：在 `CLAUDE.md` 中用 `@path/to/file` 语法引用其他文件，启动时自动展开加载（最大递归深度 5 层）。
- **`CLAUDE.local.md`**：个人项目偏好，应加入 `.gitignore`，与 `CLAUDE.md` 同时加载且优先级更高。

## 核心原则

把 `CLAUDE.md` 当作简洁的导航层，而不是文档垃圾桶。把所有任务都需要知道的规则放在 `CLAUDE.md`；把详细、条件性、子系统专用的说明放到 `.claude/rules/` 或嵌套 `CLAUDE.md`。

## 何时使用

以下请求应使用此 skill：
- 创建 / 初始化 / 编写 `CLAUDE.md`
- 更新 / 刷新 / 审计已有 agent 指令
- 创建或管理 `.claude/rules/` 规则文件
- 组织 monorepo 或包级别的 agent 指导
- 减少 agent 文档的上下文膨胀
- 创建或更新 `CLAUDE.md` 引用的文件

不要把它用于普通 README 写作，除非 README 会作为 agent 指令的来源。

## 工作流程

1. **先读再写**
   - 阅读目标范围内已有的 `CLAUDE.md` 和 `.claude/rules/` 中的规则。
   - 阅读这些文件通过 `@` 导入或引用的文件。
   - 检查项目文档和配置，用来验证命令、约定和工作流。
   - 保留明确的用户 / 项目指令，除非用户要求删除，或它们已经明显过时。

2. **选择作用域**
   - 根 `CLAUDE.md`：仓库级概览、共享命令、共享规则——每次会话都加载。
   - `.claude/rules/*.md`：按主题拆分的模块化规则——启动时自动发现。
   - `.claude/rules/*.md`（带 `paths` frontmatter）：条件加载的规则，仅在 Claude 处理匹配文件时生效。
   - 嵌套 `path/CLAUDE.md`：某个子树的本地说明——按需加载。
   - `@` 导入：在 `CLAUDE.md` 中引用 README、`AGENTS.md` 等外部文件。

3. **应用渐进披露**
   - 保持根 `CLAUDE.md` 简短、易扫读。虽然 CLAUDE.md 没有硬性长度限制，但越短的文件遵守度越高。
   - 把细节放到 `.claude/rules/` 中按主题拆分，而不是塞进根文件。
   - 使用 `paths` frontmatter 让规则只在相关文件被处理时加载，减少噪音。
   - 子系统专用说明放到嵌套 `path/CLAUDE.md`，利用按需加载机制。
   - 如果 `CLAUDE.md` 通过 `@` 导入了某个文件，要在同一次修改中创建或更新该文件。

4. **更新整套指令集**
   - 更新时确认每个被引用文件都存在。
   - 阅读并刷新本次请求影响到的每个规则文件。
   - 保留规则文件中的警告，除非它们过时、被当前事实反驳，或用户要求删除。
   - 删除过时的命令、路径和链接。
   - 修复或移除失效的 `@` 导入路径。
   - 检查根 `CLAUDE.md`、嵌套文件、规则文件之间是否存在冲突指令。

5. **验证最终文档**
   - 重新阅读已编辑的所有文件。
   - 尽可能确认命令来自真实的仓库配置或文档。
   - 确认最终结构能告诉 agent：不同任务类型应该读取哪些文件。

## 推荐的根 `CLAUDE.md` 结构

只使用适合当前仓库的章节：

```markdown
# CLAUDE.md

## 项目概览
[2-5 条要点：用途、技术栈、重要边界]

## 常用命令
- 安装：`...`
- 构建：`...`
- 测试：`...`
- Lint / 类型检查：`...`

## 工作规则
- 遵循被修改子树中的现有模式。
- 让修改范围贴合用户请求。
- 行为变化时添加或更新测试。

## 仓库地图
- `path/` — 用途；本地规则见 `path/CLAUDE.md`。
```

## 内容应该放在哪里

| 内容 | 放置位置 |
|---|---|
| 适用于仓库内所有任务的简洁规则 | 根 `CLAUDE.md` |
| 按主题拆分的详细规则（测试、安全等） | `.claude/rules/testing.md` 等 |
| 仅在处理特定文件类型时需要的规则 | `.claude/rules/*.md`（带 `paths` frontmatter） |
| 只适用于某个子目录的说明 | 嵌套 `path/CLAUDE.md` |
| 已有的外部指令（如 `AGENTS.md`、README） | 用 `@` 导入到 `CLAUDE.md` |
| 面向人的产品背景 | README 或普通文档；只摘要 agent 需要知道的部分 |

## `.claude/rules/` 规则文件

规则文件是 Claude Code 官方推荐的模块化指令组织方式。放在 `.claude/rules/` 目录中的 `.md` 文件会被自动发现。

### 无条件规则

启动时加载，适用于所有任务：

```markdown
# 测试约定

- 所有新功能必须有对应的测试
- 测试文件放在 `tests/` 目录
- 运行测试：`npm test`
```

### 条件规则（路径限定）

通过 `paths` frontmatter 限定范围，仅在 Claude 处理匹配文件时加载：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API 开发规则

- 所有端点必须有输入验证
- 使用标准错误响应格式
```

支持的 glob 模式示例：

| 模式 | 匹配 |
|---|---|
| `**/*.ts` | 所有 TypeScript 文件 |
| `src/**/*` | `src/` 下的所有文件 |
| `src/components/*.tsx` | 特定目录的 React 组件 |
| `src/**/*.{ts,tsx}` | 多扩展名匹配 |

### 推荐的规则文件

按需创建，不要预设全部：
- `.claude/rules/testing.md` — 测试约定
- `.claude/rules/frontend.md` — 前端规则
- `.claude/rules/backend.md` — 后端规则
- `.claude/rules/deployment.md` — 部署相关
- `.claude/rules/security.md` — 安全要求

## `@` 导入

`CLAUDE.md` 可以用 `@path/to/file` 语法导入其他文件，启动时自动展开：

```markdown
# CLAUDE.md

@README
@package.json
@docs/architecture.md
```

支持相对路径和绝对路径。相对路径相对于包含导入的文件解析。最大递归深度 5 层。

### 兼容 `AGENTS.md`

如果仓库已有为其他编码 agent 编写的 `AGENTS.md`，可以用 `@` 导入它而不是重复内容：

```markdown
# CLAUDE.md

@AGENTS.md

## Claude Code 专用
- 对 `src/billing/` 使用 Plan Mode
```

## 更新已有指令

当 `CLAUDE.md` 已存在时：
- 把它视为有意写下的项目上下文。
- 尽可能原样保留强指令和警告。
- 把新信息合并到现有章节，而不是为了风格重写整份文件。
- 如果把细节移动到规则文件，在原位置留下简短说明。
- 如果删除或修改了非显然的指令，在最终回复中说明。

## 常见错误

| 错误 | 更好的做法 |
|---|---|
| 把所有细节都塞进根 `CLAUDE.md` | 用 `.claude/rules/` 按主题拆分，用 `paths` 按需加载 |
| 手动维护 `docs/` 下的索引文件 | 用 `.claude/rules/`，Claude Code 自动发现 |
| 整段复制 README | 用 `@README` 导入，或只摘要 agent 需要的部分 |
| 编造命令 | 阅读配置 / 文档；无法确认时标注未知，不要猜 |
| 忽略嵌套优先级 | 子目录的 `CLAUDE.md` 在该目录被访问时才加载 |
| 刷新时删除旧指导 | 除非过时、冲突或用户要求删除，否则保留 |

## 最终回复

总结：
- 创建或修改了哪些 `CLAUDE.md`
- 创建或修改了哪些 `.claude/rules/` 规则文件
- 哪些命令或链接无法验证
- 解决了哪些指令冲突
