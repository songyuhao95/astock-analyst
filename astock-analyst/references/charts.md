# 持仓可视化

`pip install matplotlib numpy`

## 仓位分布饼图

```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams['font.family'] = 'PingFang SC'  # Windows改SimHei
matplotlib.rcParams['axes.unicode_minus'] = False

持仓 = {                  # 修改为你的持仓市值
    '半导体ETF': 20800,
    '航天电子':  20120,
    '中国西电':  19019,
    '恩捷股份':  17056,
    '化工ETF':    4590,
    '现金':       4306,
}

名称 = list(持仓.keys())
市值 = list(持仓.values())
fig, ax = plt.subplots(figsize=(8, 6))
ax.pie(市值, labels=名称, autopct='%1.1f%%', startangle=140,
       colors=['#4C8EFF','#3FB950','#F0B429','#F85149','#BC8CFF','#8B949E'],
       explode=[0.04]*len(名称), wedgeprops=dict(linewidth=1.5, edgecolor='white'))
ax.set_title(f'仓位分布  总资产¥{sum(市值):,.0f}', fontsize=14, pad=20)
plt.tight_layout()
plt.savefig('仓位分布.png', dpi=150, bbox_inches='tight')
plt.show()
```

## 持仓盈亏柱状图

```python
持仓数据 = [          # (名称, 成本价, 现价, 股数)
    ('半导体ETF', 0.919, 1.040, 20000),
    ('航天电子',  25.226, 25.150, 800),
    ('中国西电',  16.151, 17.290, 1100),
    ('恩捷股份',  87.495, 85.280, 200),
    ('化工ETF',   0.949,  1.020, 4500),
]

名称  = [d[0] for d in 持仓数据]
盈亏额 = [(d[2]-d[1])*d[3] for d in 持仓数据]
盈亏率 = [(d[2]/d[1]-1)*100 for d in 持仓数据]
颜色  = ['#F85149' if v>=0 else '#3FB950' for v in 盈亏额]

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
for ax, vals, title, fmt in [
    (ax1, 盈亏额, '持仓盈亏（元）', '¥{:+.0f}'),
    (ax2, 盈亏率, '持仓盈亏（%）', '{:+.2f}%')]:
    bars = ax.bar(名称, vals, color=颜色, width=0.5, edgecolor='white', lw=1.5)
    ax.axhline(0, color='gray', lw=0.8, ls='--')
    ax.set_title(title, fontsize=13)
    for bar, val in zip(bars, vals):
        ax.text(bar.get_x()+bar.get_width()/2,
                bar.get_height()+(max(vals, default=0)*0.03 if max(vals,default=0)>0 else 0),
                fmt.format(val), ha='center', fontsize=9)

plt.tight_layout()
plt.savefig('持仓盈亏.png', dpi=150, bbox_inches='tight')
plt.show()
```
