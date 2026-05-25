# astock-analyst

A股、ETF、大盘行情和个人持仓管理技能。

该 skill 用于在 T+5 交易窗口内整理行情、仓位、趋势、资金和风险边界，输出结构化参考建议。它不替用户做最终投资决策。

## 设计目标

- 默认主建议偏趋势进攻，同时提供稳健防守备选。
- 不因浮盈大直接建议减仓，先检查趋势、仓位上限和移动止盈线。
- 区分盘中、尾盘和收盘信号，避免把短暂破位当成正式破位。
- 先判断组合仓位，再分析单只持仓。
- 输出必须包含完整股票名称、触发条件、失效条件和主要风险。

## 文件结构

```text
astock-analyst/
  SKILL.md
  references/
    data-sources.md
    charts.md
```

## 安装

```powershell
npx skills add https://github.com/songyuhao95/astock-analyst --skill astock-analyst --agent codex -g -y --copy
```

## 免责声明

本 skill 生成内容仅用于学习、研究和辅助决策，不构成投资建议。股市有风险，最终决策和风险由使用者自行承担。
