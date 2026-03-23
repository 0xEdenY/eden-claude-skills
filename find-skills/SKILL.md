---
name: find-skills
description: 列出并搜索所有可用 skill。当用户问"有哪些 skill"、"能做什么"、"有没有关于 X 的 skill"、"查找 skill"时触发。
---

# Find Skills

列出 `~/.claude/skills/` 和 `.claude/skills/`（项目级）中所有已安装的 skill，并根据用户需求推荐合适的 skill。

## 执行步骤

1. 列出全局 skill 目录内容：`ls ~/.claude/skills/`
2. 列出项目级 skill 目录内容（若存在）：`ls .claude/skills/ 2>/dev/null`
3. 读取每个 skill 的 `SKILL.md` frontmatter，提取 `name` 和 `description`
4. 以表格形式展示所有 skill：名称 + 一句话描述 + 作用域（全局/项目）
5. 如果用户有关键词，按描述相关性排序推荐

## 输出格式

```
## 已安装的 Skills

| Skill | 描述 | 作用域 |
|-------|------|--------|
| architecture-principles | AI 时代架构设计原则 | 全局 |
| ... | ... | ... |

## 使用方法
输入 `/skill-name` 或直接描述你的需求，Claude 会自动匹配合适的 skill。
```
