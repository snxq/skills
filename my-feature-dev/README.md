# my-feature-dev

功能迭代多任务开发的编排流程：阶段规划 → 公共依赖先行 → 子 Agent 并行开发（worktree）→ 子 Agent 审查 → MR 管理。主会话是组织者与上下文管理者。

支持的功能：
- 核心约定：基于 origin 迭代基线、worktree 建分支、子 Agent 执行开发/提交/MR、并行派发、审查另起子 Agent
- 输入文档依赖（需求规格/原型/技术方案/前端接口对接文档/工作内容）与权威优先级
- 阶段规划方法（先行件/关键路径/独立件）与阻塞项清单
- 子 Agent 任务书要素模板（基线、目录、MR 约定写清楚）
- 审查子 Agent 机制与通用审查清单

## 安装

```bash
cp -r my-feature-dev ~/.claude/skills/
```

## 使用

安装后，用自然语言触发：

- `按 my-feature-dev 流程推进这个迭代`
- `把剩余任务拆成并行子 Agent 开发`
- `对子 Agent 的产出起审查子 Agent`
