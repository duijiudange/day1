# day1
a testing repostory
# -*- coding: utf-8 -*-
"""
Created on Sat Jan 06 12:58:17 2018

目的：找出某只证券在一个月中的哪一天价格最低
"""

import tushare as ts
import pandas as pd
import matplotlib.pyplot as plt 

security_code='000002'

#获取证券的后复权日行情数据
df_k=ts.get_k_data(security_code,start='1990-12-19',autype='hfq',index=False)
#提取日期和收盘价
df_c=df_k.loc[:,['date','close']]

df_c['date'] = pd.to_datetime(df_c['date']) #将数据类型转换为日期类型
df_c = df_c.set_index('date') # 将date设置为index

#转换成自然日期行标                  
d1=(df_c.index[0]).strftime('%Y%m%d')
d2=(df_c.index[-1]).strftime('%Y%m%d')

dates = pd.date_range(start=d1,end=d2) 

df_temp=pd.DataFrame('',index=dates,columns=['A'])

df_temp=pd.concat([df_temp,df_c],axis=1) # 将新表横向追加到旧表

#进行数据清洗
del df_temp['A'] #删除A列
df_temp=df_temp.fillna(0) #以0替换填充空值

#处理空值[0]：若收盘价为[0]，则当日收盘价等于后一日收盘价
r=1
while r<len(df_temp.index):
    if df_temp.iat[-r,0]==0:
        df_temp.iat[-r,0]=df_temp.iat[-(r-1),0]
    r=r+1     
                     

df_c=df_temp                 
                     
#剥离data中的“日”为标签，作为一列添加道数据表df_g,此列字段命名为"day"
df_c['day']=(df_c.index).day
#按字段"day"对数据进行分组
df_g=df_c.groupby(df_c['day'])
#对已分组的数据取平均值
df_g=df_g.mean()
#计算月中每天相对月中最低价的涨幅
df_g= ((df_g/df_g.min())-1)*100

#结论

dateToMarket=df_k.iat[0,0] #上市日期

max_v=(df_g.max())[0]#最高收盘价日相对最低收盘价日涨幅
day_of_lowest_price=(df_g[df_g.close==0]).index[0] #最低价日
day_of_highest_price=(df_g[df_g.close==max_v]).index[0] #最高价日

conclusion='证券{}自{}上市以来,在一个月中：\
{}号价格最低；{}号价格最高；{}号相对{}号上涨{:,.2f}%。\
'.format(security_code,dateToMarket,day_of_lowest_price,day_of_highest_price,\
         day_of_highest_price,day_of_lowest_price,max_v)

print conclusion


#作图
fig, ax = plt.subplots(figsize=(10, 6))   
ax.bar(df_g.index,df_g['close'])
ax.set_xticks(df_g.index)
ax.set_xlabel('Date')
ax.set_ylabel('Swing(%)')
fig.suptitle('Swing relating to lowest day in a month:'+security_code)
