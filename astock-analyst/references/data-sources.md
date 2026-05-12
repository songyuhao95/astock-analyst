# 数据获取与核验

## 多源聚合查询（推荐）

```python
import requests, json
from datetime import datetime

def validate(data: dict, code: str, name: str) -> dict:
    """三字段核验，所有数据获取后必须调用"""
    today = datetime.now().strftime("%Y-%m-%d")
    d = str(data.get("日期", data.get("date", data.get("时间", ""))))
    c = str(data.get("代码", data.get("code", data.get("symbol", ""))))
    n = str(data.get("名称", data.get("name", data.get("stockName", ""))))
    checks = [today[:10] in d or d >= today[:10],
              code in c or c.endswith(code),
              name[:2] in n or n[:2] in name]
    passed = sum(checks)
    confidence = ("✅ 已核验" if passed == 3 else
                  f"⚠️ 部分核验({passed}/3)" if passed > 0 else "❌ 未核验")
    return {"data": data, "confidence": confidence, "usable": passed == 3}


def get_em_quote(code: str, name: str) -> dict:
    """东方财富实时行情"""
    market = 1 if code.startswith(("6","5")) else 0
    url = (f"https://push2.eastmoney.com/api/qt/stock/get"
           f"?secid={market}.{code}&fields=f57,f58,f43,f44,f45,f46,f47,f48,f170,f86&cb=cb")
    field_map = {"f57":"代码","f58":"名称","f43":"最新价","f44":"最高","f45":"最低",
                 "f46":"开盘","f47":"成交量","f48":"成交额","f170":"涨跌幅","f86":"时间戳"}
    try:
        r = requests.get(url, timeout=5,
            headers={"User-Agent":"Mozilla/5.0","Referer":"https://quote.eastmoney.com"})
        raw = json.loads(r.text[r.text.index("(")+1:r.text.rindex(")")]).get("data",{})
        d = {v: raw.get(k) for k,v in field_map.items()}
        if d.get("时间戳"):
            d["日期"] = datetime.fromtimestamp(int(d["时间戳"])).strftime("%Y-%m-%d %H:%M")
        for f in ["最新价","最高","最低","开盘","涨跌额"]:
            if d.get(f): d[f] = round(d[f]/100, 3)
        if d.get("涨跌幅"): d["涨跌幅"] = round(d["涨跌幅"]/100, 2)
        return validate(d, code, name)
    except Exception as e:
        return {"confidence": f"❌ 东财失败:{e}", "usable": False}


def get_tencent_quote(code: str, name: str) -> dict:
    """腾讯证券实时行情"""
    prefix = "sh" if code.startswith(("6","5")) else "sz"
    try:
        r = requests.get(f"https://qt.gtimg.cn/q={prefix}{code}", timeout=5,
            headers={"User-Agent":"Mozilla/5.0","Referer":"https://gu.qq.com"})
        r.encoding = "gbk"
        p = r.text.split("~")
        if len(p) < 10: return {"confidence":"❌ 腾讯格式异常","usable":False}
        d = {"名称":p[1],"代码":p[2],"最新价":float(p[3]),"昨收":float(p[4]),
             "开盘":float(p[5]),"成交量":int(p[6]),
             "日期":p[30][:10] if len(p)>30 else datetime.now().strftime("%Y-%m-%d")}
        if d["最新价"] and d["昨收"]:
            d["涨跌幅"] = round((d["最新价"]/d["昨收"]-1)*100, 2)
        return validate(d, code, name)
    except Exception as e:
        return {"confidence": f"❌ 腾讯失败:{e}", "usable": False}


def get_em_kline(code: str, name: str, days: int = 60) -> dict:
    """东方财富K线（日线，前复权）"""
    market = 1 if code.startswith(("6","5")) else 0
    url = (f"https://push2his.eastmoney.com/api/qt/stock/kline/get"
           f"?secid={market}.{code}&fields1=f1,f2&fields2=f51,f52,f53,f54,f55,f56,f57"
           f"&klt=101&fqt=1&lmt={days}&cb=cb")
    try:
        r = requests.get(url, timeout=5,
            headers={"User-Agent":"Mozilla/5.0","Referer":"https://quote.eastmoney.com"})
        data = json.loads(r.text[r.text.index("(")+1:r.text.rindex(")")]).get("data",{})
        import pandas as pd
        records = [dict(zip(["日期","开盘","收盘","最高","最低","成交量","成交额"],
                            [k.split(",")[0]]+[float(x) if i<5 else int(float(x))
                             for i,x in enumerate(k.split(",")[1:7])]))
                   for k in data.get("klines",[])]
        df = pd.DataFrame(records)
        v = validate({"日期":df.iloc[-1]["日期"],"代码":code,"名称":name}, code, name)
        v["dataframe"] = df
        return v
    except Exception as e:
        return {"confidence": f"❌ 东财K线失败:{e}", "usable": False, "dataframe": None}


def get_akshare_quote(code: str, name: str) -> dict:
    """akshare兜底，自动附时效提示"""
    import akshare as ak
    try:
        df = ak.stock_zh_a_spot_em()
        row = df[df['代码']==code]
        if row.empty: return {"confidence":"❌ akshare未找到","usable":False}
        d = row.iloc[0].to_dict()
        d["日期"] = datetime.now().strftime("%Y-%m-%d")
        r = validate(d, code, name)
        r["confidence"] += " 🕐akshare兜底，建议交叉确认"
        return r
    except Exception as e:
        return {"confidence": f"❌ akshare失败:{e}", "usable": False}


def get_quote(code: str, name: str) -> dict:
    """多源聚合实时行情，按优先级尝试"""
    for src, fn in [("东方财富", lambda: get_em_quote(code, name)),
                    ("腾讯行情", lambda: get_tencent_quote(code, name)),
                    ("akshare",  lambda: get_akshare_quote(code, name))]:
        r = fn()
        if r.get("usable"):
            print(f"✅ 来源:{src} {r['confidence']}")
            return r
    print("❌ 所有来源均未核验通过，建议前往交易软件手动确认")
    return {"usable": False}


def get_kline(code: str, name: str, days: int = 60):
    """多源聚合K线"""
    import akshare as ak, pandas as pd
    from datetime import timedelta
    r = get_em_kline(code, name, days)
    if r.get("usable") and r.get("dataframe") is not None:
        print(f"✅ K线来源:东财 最新:{r['dataframe'].iloc[-1]['日期']}")
        return r["dataframe"]
    # akshare兜底
    try:
        df = ak.stock_zh_a_hist(symbol=code, period="daily", adjust="qfq").tail(days).reset_index(drop=True)
        latest = str(df.iloc[-1]["日期"])
        ok = latest >= (datetime.now()-timedelta(days=2)).strftime("%Y-%m-%d")
        print(f"{'✅' if ok else '🕐'} K线来源:akshare 最新:{latest}")
        return df
    except:
        print("❌ K线获取失败"); return pd.DataFrame()
```

## 资金流向（akshare）

```python
import akshare as ak
from datetime import datetime

def get_money_flow(code: str, name: str, market: str = "sh") -> dict:
    """近5日主力资金流向，market: sh/sz"""
    try:
        df = ak.stock_individual_fund_flow(stock=code, market=market)
        df = df.tail(5)[['日期','主力净流入-净额','主力净流入-净占比','超大单净流入-净额']]
        latest = str(df.iloc[-1]['日期'])
        ok = latest >= datetime.now().strftime("%Y-%m-%d")[:8]
        print(f"{'✅' if ok else '🕐 时效存疑'} 资金流向 代码:{code} 名称:{name} 最新:{latest}")
        return {"dataframe": df, "usable": ok}
    except Exception as e:
        return {"confidence": f"❌ 资金流向失败:{e}", "usable": False}
```
