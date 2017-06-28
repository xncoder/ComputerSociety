# 策略
## 等权PE-PB计算
```
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import bisect

#指定日期的指数PE（等权重）
def get_index_pe_date(index_code,date):
    stocks = get_index_stocks(index_code, date)
    q = query(valuation).filter(valuation.code.in_(stocks))
    df = get_fundamentals(q, date)
    if len(df)>0:
        pe = len(df)/sum([1/p if p>0 else 0 for p in df.pe_ratio])
        return pe
    else:
        return float('NaN')

#指定日期的指数PB（等权重）
def get_index_pb_date(index_code,date):
    stocks = get_index_stocks(index_code, date)
    q = query(valuation).filter(valuation.code.in_(stocks))
    df = get_fundamentals(q, date)
    if len(df)>0:
        pb = len(df)/sum([1/p if p>0 else 0 for p in df.pb_ratio])
        return pb
    else:
        return float('NaN')
    
#指数历史PEPB
def get_index_pe_pb(index_code):
    start='2005-1-1'
    #start='2015-1-1'
    end = pd.datetime.today();
    dates=[]
    pes=[]
    pbs=[]
    for d in pd.date_range(start,end,freq='M'): #频率为月
        dates.append(d)
        pes.append(get_index_pe_date(index_code,d))
        pbs.append(get_index_pb_date(index_code,d))
    d = {'PE' : pd.Series(pes, index=dates),
            'PB' : pd.Series(pbs, index=dates)}
    PB_PE = pd.DataFrame(d)
    return PB_PE


all_index = get_all_securities(['index'])
index_choose =['000016.XSHG',                        
                         '000300.XSHG',
                         '000902.XSHG',
                         '000905.XSHG',
                         '399106.XSHE',               
                         '399316.XSHE',
               
                        '000036.XSHG',
                        '000037.XSHG',
                        '000038.XSHG',
                        '000039.XSHG',
                        '000158.XSHG'
                        ]
df_pe_pb = pd.DataFrame()
frames=pd.DataFrame()
today= pd.datetime.today()
for code in index_choose:
    index_name = all_index.ix[code].display_name  
    print u'正在处理: ',index_name   
    df_pe_pb=get_index_pe_pb(code)    

    
    
    
    
    results=[]
    pe = get_index_pe_date(code,today)
    q_pes = [df_pe_pb['PE'].quantile(i/10.0)  for i in range(11)]    
    idx = bisect.bisect(q_pes,pe)
    quantile = idx-(q_pes[idx]-pe)/(q_pes[idx]-q_pes[idx-1])   
    #index_name = all_index.ix[code].display_name
    results.append([index_name,'%.2f'% pe,'%.2f'% (quantile*10)]+['%.2f'%q  for q in q_pes]+[df_pe_pb['PE'].count()])
    
    pb = get_index_pb_date(code,today)
    q_pbs = [df_pe_pb['PB'].quantile(i/10.0)  for i in range(11)] 
    idx = bisect.bisect(q_pbs,pb)
    quantile = idx-(q_pbs[idx]-pb)/(q_pbs[idx]-q_pbs[idx-1])   
    #index_name = all_index.ix[code].display_name
    results.append([index_name,'%.2f'% pb,'%.2f'% (quantile*10)]+['%.2f'%q  for q in q_pbs]+[df_pe_pb['PB'].count()])
    
    
    df_pe_pb['10% PE']=q_pes[1]
    df_pe_pb['50% PE']=q_pes[5]
    df_pe_pb['90% PE']=q_pes[9]
    df_pe_pb['10% PB']=q_pbs[1]
    df_pe_pb['50% PB']=q_pbs[5]
    df_pe_pb['90% PB']=q_pbs[9]

    df_pe_pb.plot(secondary_y=['PB','10% PB','50% PB','90% PB'],figsize=(14,8),title=index_name,style=['k-.', 'k', 'g', 'y', 'r', 'g-.', 'y-.', 'r-.']) 
    columns=[u'名称',u'当前估值',u'分位点%',u'最小估值']+['%d%%'% (i*10) for i in range(1,10)]+[u'最大估值' , u"数据个数"]
    df= pd.DataFrame(data=results,index=['PE','PB'],columns=columns)
    frames = pd.concat([frames, df])
frames  
```