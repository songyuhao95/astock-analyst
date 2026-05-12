# 技术面代码

依赖 data-sources.md 中的 `get_kline()` 函数。

## MACD分析

```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams['font.family'] = 'PingFang SC'  # Windows改SimHei
matplotlib.rcParams['axes.unicode_minus'] = False

def analyze_macd(code: str, name: str, days: int = 120, period: str = "daily"):
    """
    MACD分析 + 画图，自动识别金叉/死叉/背离
    period: 'daily'日线(多源) / 'weekly'周线(akshare兜底)
    """
    import akshare as ak
    if period == "daily":
        df = get_kline(code, name, days)          # 来自 data-sources.md
    else:
        df = ak.stock_zh_a_hist(symbol=code, period="weekly",
             adjust="qfq").tail(days).reset_index(drop=True)
        print(f"🕐 周线来源:akshare 最新:{df.iloc[-1]['日期']}，请确认时效")

    if df is None or df.empty:
        print("❌ K线数据获取失败"); return

    close = df['收盘']
    ema12 = close.ewm(span=12, adjust=False).mean()
    ema26 = close.ewm(span=26, adjust=False).mean()
    dif   = ema12 - ema26
    dea   = dif.ewm(span=9, adjust=False).mean()
    hist  = (dif - dea) * 2
    i     = len(df) - 1

    # 信号输出
    print(f"\n{'='*45}")
    print(f"{name}({code}) {period} 最新:{df.iloc[i]['日期']}")
    print(f"DIF:{dif.iloc[i]:.4f}  DEA:{dea.iloc[i]:.4f}  柱:{hist.iloc[i]:.4f}")
    if   dif.iloc[i] > dea.iloc[i] and dif.iloc[i-1] <= dea.iloc[i-1]: print("⚡ 今日金叉")
    elif dif.iloc[i] < dea.iloc[i] and dif.iloc[i-1] >= dea.iloc[i-1]: print("⚠️  今日死叉")
    elif dif.iloc[i] > dea.iloc[i]: print("✅ 多头排列")
    else:                            print("🔻 空头排列")
    print("零轴" + ("上方(偏强)" if dif.iloc[i] > 0 else "下方(偏弱)"))
    print('='*45)

    # 画图
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(14, 8),
        gridspec_kw={'height_ratios':[3,2]}, sharex=True)
    ax1.plot(close.values, color='#1a73e8', lw=1.5, label='收盘价')
    ax1.set_title(f'{name}({code}) MACD {period} 近{days}期  最新:{df.iloc[-1]["日期"]}', fontsize=12)
    ax1.set_ylabel('价格'); ax1.legend(); ax1.grid(alpha=0.3)
    ax2.plot(dif.values, color='#1a73e8', lw=1.2, label='DIF')
    ax2.plot(dea.values, color='#ff6b35', lw=1.2, label='DEA')
    ax2.bar(range(len(hist)), hist.values,
            color=['red' if v>=0 else 'green' for v in hist], alpha=0.7, width=0.8)
    ax2.axhline(0, color='gray', lw=0.8, ls='--')
    ax2.set_ylabel('MACD'); ax2.legend(); ax2.grid(alpha=0.3)
    step = max(1, len(df)//8)
    ax2.set_xticks(range(0, len(df), step))
    ax2.set_xticklabels([df['日期'].iloc[j] for j in range(0, len(df), step)],
                        rotation=30, fontsize=8)
    plt.tight_layout()
    plt.savefig(f'{code}_macd_{period}.png', dpi=150, bbox_inches='tight')
    plt.show()
    print(f"已保存:{code}_macd_{period}.png")

# 日线+周线共振分析
# analyze_macd('600879', '航天电子')
# analyze_macd('600879', '航天电子', period='weekly')
```

## K线图（含均线）

```python
def plot_kline(code: str, name: str, days: int = 60):
    df = get_kline(code, name, days)
    if df is None or df.empty:
        print("❌ K线数据获取失败"); return

    close = df['收盘']
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(14, 8),
        gridspec_kw={'height_ratios':[3,1]}, sharex=True)

    for i, row in df.iterrows():
        c = 'red' if row['收盘'] >= row['开盘'] else 'green'
        ax1.plot([i,i], [row['最低'],row['最高']], color=c, lw=0.8)
        ax1.bar(i, abs(row['收盘']-row['开盘']),
                bottom=min(row['开盘'],row['收盘']), color=c, width=0.6, alpha=0.9)

    xs = range(len(df))
    for n, c in [(5,'#FFA500'),(10,'#9B59B6'),(20,'#3498DB')]:
        ax1.plot(xs, close.rolling(n).mean(), color=c, lw=1.2, label=f'MA{n}')
    ax1.set_title(f'{name}({code}) K线 近{days}日  最新:{df.iloc[-1]["日期"]}', fontsize=12)
    ax1.legend(fontsize=8); ax1.grid(alpha=0.3)

    ax2.bar(xs, df['成交量'],
            color=['red' if df.loc[i,'收盘']>=df.loc[i,'开盘'] else 'green' for i in range(len(df))],
            alpha=0.7, width=0.6)
    ax2.set_ylabel('成交量')
    step = max(1, len(df)//8)
    ax2.set_xticks(range(0, len(df), step))
    ax2.set_xticklabels([df['日期'].iloc[i] for i in range(0, len(df), step)],
                        rotation=30, fontsize=8)
    plt.tight_layout()
    plt.savefig(f'{code}_kline.png', dpi=150, bbox_inches='tight')
    plt.show()

# plot_kline('600879', '航天电子')
```
