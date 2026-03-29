---
name: consistency-guard
description: 文件一致性校验。当用户说"检查一致性"、"文件对齐吗"、"依赖校验"、"一致性审计"时触发。
---

# 文件一致性校验

项目文件（PRD、tasks.md、code、CLAUDE.md、progress.md）之间有依赖关系。上游变了下游可能需要更新。本 Skill 管理这些依赖的可见性和一致性。

不变原则：
- 依赖单向（PRD → tasks → code → tests，不反向）
- 边界即契约（每类文档对下游有明确承诺）
- 变更传播最小化（只校验受影响的下游，不全量扫描）

## 三个工作流

| 工作流 | 触发词 | 做什么 |
|--------|--------|--------|
| W1 生成依赖地图 | "生成依赖地图"、"更新依赖地图" | 扫描项目文件，生成 docs/dependency-map.md |
| W2 一致性校验 | "检查一致性"、"依赖校验" | 按地图逐条校验，级联检查，产出报告 |
| W3 偏离标记 | "标记偏离" | 在上游文件插入 DIVERGENCE 标记 |

---

## W1：生成/更新依赖地图

1. 扫描 docs/ 目录、CLAUDE.md 文件、src/ 目录
2. 识别文件间引用关系（grep 交叉引用、文件名互引）
3. 生成或更新 `docs/dependency-map.md`，结构如下：

```markdown
# 依赖地图

## 依赖方向（DAG）

PRD → tasks.md → code → tests
PRD → CLAUDE.md（项目级技术约束）
tasks.md → progress.md（完成状态）
CLAUDE.md 全局 → CLAUDE.md 项目 → CLAUDE.md 目录

## 校验规则

| 上游 | 下游 | 校验规则 | 级别 |
|------|------|---------|------|
| docs/prd.md | docs/tasks.md | 每条需求至少一个 Task 覆盖 | L2 |
| docs/prd.md | src/** | 已实现功能行为与 PRD 一致 | L2 |
| docs/tasks.md | docs/tasks.md | 依赖关系与完成顺序一致 | L2 |
| docs/tasks.md | src/** | 标记完成的 Task 功能存在于代码 | L2 |
| CLAUDE.md 各层 | CLAUDE.md 各层 | 无直接矛盾 | L2 |
| docs/progress.md | git log | 完成标记与 git 一致 | L2 |
| 任意上游 | 其下游链 | 确定性不一致时检查下游链 | L3 |

## DIVERGENCE 记录

_(W2 运行时自动汇总)_

## Baseline

_(首次 W2 运行时生成)_
```

4. 展示 DAG，让用户确认后写入文件
5. 校验规则行可按项目需要增删

---

## W2：一致性校验

1. 读取 `docs/dependency-map.md`（不存在则先触发 W1）
2. 首次运行：生成 baseline（当前状态快照），存入 Baseline section
3. 对每条校验规则：
   - 读取上下游文件
   - 判断一致/不一致
   - 标注检查类型：**确定性**（文件存在、状态匹配、git 对账）或**语义**（内容一致性）
4. 扫描所有文件中的 `DIVERGENCE` 标记，汇总
5. 级联（L3）：确定性不一致 → 沿 DAG 继续检查下游链；语义不一致 → 只单独报告，不级联
6. 产出一致性报告

**报告格式：**

```markdown
## 一致性报告 [日期]

### ✅ 一致（确定性）
- [校验项]: 结果

### ✅ 一致（语义，供参考）
- [校验项]: 结果

### ⚠️ 已标记偏离
- [文件] [DIVERGENCE Task-X]: 内容
  → 修复建议：更新上游 / 扩展实现

### ❌ 未标记不一致
- [校验项]: 不一致描述（确定性/语义）
  → 修复建议
  → 级联检查结果（仅确定性）

### 📊 级联影响
- [上游不一致] → [下游受影响项] → 建议
```

---

## W3：偏离标记

1. 用户说明偏离内容和原因
2. 在上游文件对应位置插入：

```
> ⚠️ DIVERGENCE [Task-X, 日期]: 偏离内容。原因：xxx
```

3. 标记是临时的——W2 报告会提示处理，处理后删除标记

---

## 能力级别

| 级别 | 能力 | 何时启用 |
|------|------|---------|
| L0 | DIVERGENCE 即时标记 | 所有项目 |
| L1 | dependency-map 可视化 | 所有项目 |
| L2 | 一致性校验（逐条检查） | 中型项目+ |
| L3 | 级联影响评估 | 大型项目 |
| L4 | product-iteration 前 gate | 大型项目 |

小项目只需 L0-L1（标记 + 看见依赖）。大型项目启用 L2-L4（检测 + 级联 + gate）。

---

## 与其他 Skill 的协作

- **product-builder**：阶段 5 结束后触发 W1 生成初始依赖地图
- **dev-workflow**：阶段 4 发现偏离时触发 W3 标记
- **change-guard**：W4 文档同步后触发 W2 校验
- **product-iteration**：开始前触发 W2 作为 gate（不一致状态下不做迭代决策）
