# 基本面与机构代码

## 财报与估值

```python
import akshare as ak

def get_financials(code: str, exchange: str = "sh"):
    """近8期利润表。exchange: sh/sz"""
    df = ak.stock_financial_report_sina(stock=f"{exchange}{code}", symbol="利润表")
    cols = ['报告期','营业收入','净利润','扣除非经常性损益后的净利润']
    return df[[c for c in cols if c in df.columns]].head(8)

def get_valuation(code: str):
    """PE/PB/ROE近8期"""
    df = ak.stock_a_indicator_lg(symbol=code)
    return df[['trade_date','pe','pb','ps','roe']].tail(8)

def get_forecast(date: str = "20251"):
    """业绩预告。date格式: 年份+季度 如'20251'=2025一季度"""
    return ak.stock_yjyg_em(date=date).head(30)
```

## 解禁减持

```python
def get_unlocking(code: str):
    """个股限售股解禁计划"""
    return ak.stock_restricted_release_detail_em(symbol=code).head(10)

def get_unlock_calendar():
    """全市场近期解禁日历"""
    return ak.stock_restricted_release_queue_em().head(30)
```

## 机构多空

```python
def get_billboard(code: str = None):
    """龙虎榜近一月。code指定则过滤个股"""
    df = ak.stock_lhb_detail_em(symbol="近一月")
    return df[df['代码']==code].head(20) if code else df.head(30)

def get_north_flow(days: int = 5):
    """北向资金净流入近N日"""
    df = ak.stock_hsgt_north_net_flow_in_em(symbol="沪深港通")
    return df.tail(days)[['日期','当日成交净买额']]

def get_north_holding(code: str):
    """个股北向持仓"""
    return ak.stock_hsgt_individual_em(symbol=code).tail(10)

def get_margin(code: str, days: int = 10):
    """个股融资融券近N日"""
    df = ak.stock_margin_detail_em(symbol=code)
    return df.tail(days)[['日期','融资余额','融资买入额','融券余量','融券卖出量']]

def get_block_trade(code: str, start: str = "20250101", end: str = "20251231"):
    """个股大宗交易"""
    df = ak.stock_dzjy_mrmx(type_="DBQ", start_date=start, end_date=end)
    return df[df['证券代码']==code].head(10)
```

⚠️ 以上均为 akshare 数据，使用时须核验最新日期与目标代码/名称是否一致。
