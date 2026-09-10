---
name: dispatching-parallel-agents
description: Use when two or more agents can work concurrently, including tasks that touch the same file or logical area and need coordinated patch integration
---

# 并行代理调度

并行的目标是缩短独立路径；主 Agent 保留需求解释、共享接口、整合和最终验证。**同一文件可以被多个 Agent 并行处理，但任何变更进入共享最终工作区前，都必须带可靠基线并经过协调器合并。**不得无记录地直接覆盖共享最终工作区。

## 任务分类

- **调查**：只读收集事实、复现步骤、测试地图或风险。
- **实现**：基于协调器提供的共同基线产出独立变更集。
- **审查**：只读检查实现或变更集，返回可复现 finding，不顺手修改实现。
- **整合**：在隔离副本中应用变更集、处理冲突并验证结果。

## 重叠判断

| 重叠关系 | 调度方式 |
|---|---|
| 文件和逻辑区域都不同 | 直接并行 |
| 同一文件、不同可定位区域 | 并行生成 patch，再读取 `references/overlap-integration.md` 合并 |
| 同一文件、同一区域 | 可并行分析；实现 patch 进入合并队列 |
| 共享接口、迁移或配置契约 | 主 Agent 先定接口/不变量，再并行实现；必要时改为顺序 |

重叠不自动等于不能并行；没有共同基线、变更不可重放、无法判定验收或动作不可逆且未获授权时才停止。文本合并成功和最小测试通过都不构成 `validated`：先查不变量与同名共享符号，语义审查过了再判状态。

## 调度边界

默认只由主 Agent 向下分派一层。只有整合任务本身独立、可验收且不继续争用同一变更集时，才允许有限递归；当剩余工作主要是整合、互相等待或冲突仲裁时，停止继续拆分。任务很小且调度、等待和整合成本明显高于工作量时，主 Agent 直接处理。

## 按需读取的协议

- 派发 Agent、填写任务输入或返回结果时，读取 [`references/dispatch-contract.md`](references/dispatch-contract.md)。
- 出现同文件、同区域、共享接口、patch 合并、冲突或验证回退时，读取 [`references/overlap-integration.md`](references/overlap-integration.md)。
- 只读调查且没有文件重叠时，不需要加载重叠合并协议。

## 最终责任

子 Agent 的报告、patch 应用成功或文本无冲突都不是完成证据。主 Agent 必须检查实际 diff、接口兼容性、未解决风险和最终验证结果；提交、推送、删除、生产或其他不可逆外部写入仍按用户授权单独确认。

## 常见错误

- 没有 patch、基线或验证结果却声称完成：读取变更集契约并标记 `blocked`。
- 把文本可合并或最小测试通过当成 `validated`：读取重叠合并协议，按判定优先级先做语义审查。
- 让冲突后的 Agent 互相覆盖：保留双方变更，交给隔离的整合流程。
- 为了并行拆开共享契约：先由主 Agent 决策契约，再并行实现。
- 把子 Agent 的"合并建议"当成协调器状态：建议只是输入，状态由协议判定。
