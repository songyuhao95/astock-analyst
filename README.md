# Agent Skills

这个仓库用于存放我自定义的 Agent Skills。当前包含：

- [`astock-analyst`](./astock-analyst)：A 股市场分析与个人持仓管理技能

## astock-analyst

`astock-analyst` 是一个面向 A 股、ETF、大盘行情和个人持仓分析的 Agent Skill。它会把技术面、基本面、政策和宏观事件放到同一个 T+5 交易日窗口中，帮助智能体输出更稳定、更结构化的分析结果。

适用场景：

- 上传持仓截图或持仓数据后，提取股票代码、成本、持股数、盈亏和仓位
- 询问单只 A 股、ETF、行业板块或大盘的短期走势
- 分析加仓、减仓、止损、止盈和持有时机
- 对浮亏、套牢、仓位过重等情况做 T+5 操作建议
- 结合财报、机构动向、政策变化和宏观事件判断短期风险
- 生成持仓分析图表或相关代码

## 输出特点

该技能要求智能体按固定结构输出：

1. 持仓提取表
2. T+5 信号分析表
3. T+5 操作建议表
4. 三句话总结

技能强调数据来源、日期和置信度，避免把过期数据或无法核实的数据直接作为决策依据。

## 安装

### 使用 Skills CLI

安装到全局 skills：

```powershell
npx skills add https://github.com/songyuhao95/astock-analyst --skill astock-analyst -g -y --copy
```

安装到 Codex：

```powershell
npx skills add https://github.com/songyuhao95/astock-analyst --skill astock-analyst --agent codex -g -y --copy
```

安装到 Claude Code：

```powershell
npx skills add https://github.com/songyuhao95/astock-analyst --skill astock-analyst --agent claude-code -g -y --copy
```

### 手动安装到 Claude Code

Claude Code 的个人 skills 目录通常是：

```text
~/.claude/skills/<skill-name>/SKILL.md
```

Windows PowerShell 示例：

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
git clone https://github.com/songyuhao95/astock-analyst.git "$env:TEMP\agent-skills"
Copy-Item "$env:TEMP\agent-skills\astock-analyst" "$env:USERPROFILE\.claude\skills" -Recurse -Force
```

重启 Claude Code 后，可用以下方式调用：

```text
/astock-analyst
```

## 使用示例

```text
使用 astock-analyst 技能，帮我分析 600519 未来 T+5 的操作建议。
```

```text
这是我的持仓截图，请按 astock-analyst 的三张表格式分析，并给出加仓、减仓、持有或止损建议。
```

```text
帮我看一下今天大盘位置，以及我的科技 ETF 是否适合继续持有。
```

## 数据来源说明

技能优先要求智能体使用实时搜索和公开数据源核验行情、财报、政策和宏观事件。不同智能体的联网能力不同，如果当前环境不能联网，分析结果应标注为待核实或仅供参考。

## 免责声明

本技能生成的内容仅用于学习、研究和辅助决策，不构成任何投资建议。股市有风险，投资需谨慎，最终决策和风险由使用者自行承担。
