# Prompt 骨架模板

从零写 prompt 时的起点。按需裁剪，不要全部照搬。

## XML 分区版（适合 Claude 长 system prompt）

```xml
<role>
你是 [角色]。你的任务是 [一句话目标]。
</role>

<instructions>
[编号列表，每条一个明确指令]
</instructions>

<constraints>
[禁止清单 / 边界条件 / 格式要求]
</constraints>

<output_format>
[精确的输出格式定义，用示例说明]
</output_format>

<examples>
[2-3 个：正例 + 边界 + 反例]
</examples>
```

## Markdown 版（适合 GPT / 短 prompt）

```markdown
# 角色与目标
[一句话]

## 指令
1. ...
2. ...

## 约束
- ...

## 输出格式
[示例]
```

## 极简版（适合小模型 / 成本敏感场景）

few-shot 比指令更有效。直接给 2-3 个输入→输出示例，最后加一个待处理输入。
