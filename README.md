## 小型金融知识图谱构流程示范

![](https://img.shields.io/static/v1?label=Author&message=jm199504&color=green)
![](https://img.shields.io/static/v1?label=Languages&message=python3.6+&color=orange)
![](https://img.shields.io/static/v1?label=LastUpdateTime&message=2020.09.30+&color=lightgrey)

[TOC]

## 1 知识图谱存储方式

主要包含资源描述框架(Resource Description Framework，RDF)和图数据库，其中RDF有以下特性：

- 存储为三元组（Triple）

- 标准的推理引擎

- W3C标准

- 易于发布数据

- 多数为学术界场景

图数据库有以下特性：

- 节点和关系均可以包含属性

- 没有标准的推理引擎

- 图的遍历效率高

- 事务管理

- 多数为工业界场景

---

## 2 关于neo4j安装、启动、登录

### 2.1 下载

[下载链接](https://neo4j.com/download-center/)

### 2.2 启动

进入neo4j目录

```
cd neo4j/bin
./neo4j start
```

启动成功，终端会提示：

```
Starting Neo4j.Started neo4j (pid 30914). It is available at http://localhost:7474/ There may be a short delay until the server is ready.
```

#### 2.2.1 打开 http://localhost:7474

#### 2.2.2 初始账户和密码均为neo4j（host类型选择bolt）

#### 2.2.3 输入旧密码并输入新密码

启动前注意本地已安装JDK（建议安装JDK版本11）：https://www.oracle.com/java/technologies/javase-downloads.html

完成安装JDK1.8.0_261后，在启动neo4j过程中出现了以下问题：

```
Unable to find any JVMs matching version "11"
```

解决：提示安装jdk 11 version，于是下载了jdk-11.0.8，Mac OS可通过`ls -la /Library/Java/JavaVirtualMachines/`查看已安装的jdk及版本信息。

#### 2.2.3 登录

---

## 3. 知识图谱构建流程

### 3.1 数据接口

免费开源金融数据接口：

a. Tushare: [http://www.tushare.org](http://www.tushare.org/)

b. JointQuant: https://www.joinquant.com/

```python
import tushare as ts
import csv
import time
import pandas as pd
# 以下pro_api token可能已过期，可自行前往申请或者使用免费版本
pro = ts.pro_api('4340a981b3102106757287c11833fc14e310c4bacf8275f067c9b82d')
```



### 3.2 数据获取

#### 3.2.1 股票基本信息

```python
stock_basic = pro.stock_basic(list_status='L', fields='ts_code, symbol, name, industry')
# 重命名行，便于后面导入neo4j
basic_rename = {'ts_code': 'TS代码', 'symbol': '股票代码', 'name': '股票名称', 'industry': '行业'}
stock_basic.rename(columns=basic_rename, inplace=True)
# 保存为stock_basic.csv
stock_basic.to_csv('financial_data\\stock_basic.csv', encoding='gbk')
```



#### 3.2.2 股票持有股东信息    

```python
holders = pd.DataFrame(columns=('ts_code', 'ann_date', 'end_date', 'holder_name', 'hold_amount', 'hold_ratio'))
# 获取一年内所有上市股票股东信息（可以获取一个报告期的）
for i in range(3610):
   code = stock_basic['TS代码'].values[i]
   holders = pro.top10_holders(ts_code=code, start_date='20180101', end_date='20181231')
   holders = holders.append(holders)
   if i % 600 == 0:
       print(i)
   time.sleep(0.4)# 数据接口限制
# 保存为stock_holders.csv
holders.to_csv('financial_data\\stock_holders.csv', encoding='gbk')
holders = pro.holders(ts_code='000001.SZ', start_date='20180101', end_date='20181231')
```



#### 3.2.3 股票概念信息   

```python
concept_details = pd.DataFrame(columns=('id', 'concept_name', 'ts_code', 'name'))
for i in range(358):
   id = 'TS' + str(i)
   concept_detail = pro.concept_detail(id=id)
   concept_details = concept_details.append(concept_detail)
   time.sleep(0.4)
# 保存为concept_detail.csv
concept_details.to_csv('financial_data\\stock_concept.csv', encoding='gbk')
```



#### 3.2.4 股票公告信息   

```python
for i in range(3610):
   code = stock_basic['TS代码'].values[i]
   notices = pro.anns(ts_code=code, start_date='20180101', end_date='20181231', year='2018')
   notices.to_csv("financial_data\\notices\\"+str(code)+".csv",encoding='utf_8_sig',index=False)
notices = pro.anns(ts_code='000001.SZ', start_date='20180101', end_date='20181231', year='2018')
```



#### 3.2.5 财经新闻信息

```python
news = pro.news(src='sina', start_date='20180101', end_date='20181231')
news.to_csv("financial_data\\news.csv",encoding='utf_8_sig')
```



#### 3.2.6 概念信息   

```python
concept = pro.concept()
concept.to_csv('financial_data\\concept.csv', encoding='gbk')
```



#### 3.2.7 沪股通和深股通成分信息

```python
#获取沪股通成分
sh = pro.hs_const(hs_type='SH')
sh.to_csv("financial_data\\sh.csv",index=False)
#获取深股通成分
sz = pro.hs_const(hs_type='SZ')
sz.to_csv("financial_data\\sz.csv",index=False)
```



#### 3.2.8 股票价格信息

```python
for i in range(3610):
   code = stock_basic['TS代码'].values[i]
   price = pro.query('daily', ts_code=code, start_date='20180101', end_date='20181231')
   price.to_csv("financial_data\\price\\"+str(code)+".csv",index=False)
```



#### 3.2.9 其它类型数据（以`tushare`免费接口为例）

```python
# 基本面信息
df = ts.get_stock_basics()
# 公告信息
ts.get_notices("000001")
# 新浪股吧
ts.guba_sina()
# 历史价格数据
ts.get_hist_data("000001")
# 历史价格数据（周粒度）
ts.get_hist_data("000001",ktype="w")
# 历史价格数据（1分钟粒度）
ts.get_hist_data("000001",ktype="m")
# 历史价格数据（5分钟粒度）
ts.get_hist_data("000001",ktype="5")
# 指数数据（sh上证指数;sz深圳成指;hs300沪深300;sz50上证50;zxb中小板指数;cyb创业板指数）
ts.get_hist_data("cyb")
# 宏观数据(居民消费指数)
ts.get_cpi()
# 获取分笔数据
ts.get_tick_data('000001', date='2018-10-08', src='tt')
```



### 3.3 数据预处理

#### 3.3.1 统计股票的交易日量众数

```python
import numpy as np

yaxis = list()
for i in listdir:
    stock = pd.read_csv("financial_data\\price_logreturn\\"+i)
    yaxis.append(len(stock['logreturn']))
counts = np.bincount(yaxis)

np.argmax(counts)
```



#### 3.3.2 计算股票对数收益   

```python
import pandas as pd
import numpy as np
import os
import math

listdir = os.listdir("financial_data\\price")

for l in listdir:
   stock = pd.read_csv('financial_data\\price\\'+l)
   stock['index'] = [1]* len(stock['close'])
   stock['next_close'] = stock.groupby('index')['close'].shift(-1)
   stock = stock.drop(index=stock.index[-1])
   logreturn = list()
   for i in stock.index:
       logreturn.append(math.log(stock['next_close'][i]/stock['close'][i]))
   stock['logreturn'] = logreturn
   stock.to_csv("financial_data\\price_logreturn\\"+l,index=False)
```

#### 3.3.3 股票间对数收益率相关性

```python
from math import sqrt
def multipl(a,b):
   sumofab=0.0
   for i in range(len(a)):
       temp=a[i]*b[i]
       sumofab+=temp
   return sumofab

def corrcoef(x,y):
   n=len(x)
   #求和
   sum1=sum(x)
   sum2=sum(y)
   #求乘积之和
   sumofxy=multipl(x,y)
   #求平方和
   sumofx2 = sum([pow(i,2) for i in x])
   sumofy2 = sum([pow(j,2) for j in y])
   num=sumofxy-(float(sum1)*float(sum2)/n)
   #计算皮尔逊相关系数
   den=sqrt((sumofx2-float(sum1**2)/n)*(sumofy2-float(sum2**2)/n))
   return num/den
```

由于原始数据达百万条，为节省计算量仅选取前300个股票进行关联性分析

```python
listdir = os.listdir("financial_data\\300stock_logreturn")
s1 = list()
s2 = list()
corr = list()
for i in listdir:
   for j in listdir:
       stocka = pd.read_csv("financial_data\\300stock_logreturn\\"+i)
       stockb = pd.read_csv("financial_data\\300stock_logreturn\\"+j)
       if len(stocka['logreturn']) == 242 and len(stockb['logreturn']) == 242:
           s1.append(str(i)[:10])
           s2.append(str(j)[:10])
           corr.append(corrcoef(stocka['logreturn'],stockb['logreturn']))
           print(str(i)[:10],str(j)[:10],corrcoef(stocka['logreturn'],stockb['logreturn']))
corrdf = pd.DataFrame()
corrdf['s1'] = s1
corrdf['s2'] = s2
corrdf['corr'] = corr
corrdf.to_csv("financial_data\\corr.csv")
```



### 3 搭建金融知识图谱

### 3.1 连接

具体代码可参考3.1 python操作neo4j-连接

```python
from pandas import DataFrame
from py2neo import Graph,Node,Relationship,NodeMatcher
import pandas as pd
import numpy as np
import os
# 连接Neo4j数据库
graph = Graph('http://localhost:7474/db/data/',username='neo4j',password='neo4j')
```

### 3.2 读取数据

```python
stock = pd.read_csv('stock_basic.csv',encoding="gbk")
holder = pd.read_csv('holders.csv')
concept_num = pd.read_csv('concept.csv')
concept = pd.read_csv('stock_concept.csv')
sh = pd.read_csv('sh.csv')
sz = pd.read_csv('sz.csv')
corr = pd.read_csv('corr.csv')
```

### 3.3 填充和去重

```python
stock['行业'] = stock['行业'].fillna('未知')
holder = holder.drop_duplicates(subset=None, keep='first', inplace=False)
```

### 3.4 创建实体

概念、股票、股东、股通

```python
sz = Node('深股通',名字='深股通')
graph.create(sz)  

sh = Node('沪股通',名字='沪股通')
graph.create(sh)  

for i in concept_num.values:
   a = Node('概念',概念代码=i[1],概念名称=i[2])
   print('概念代码:'+str(i[1]),'概念名称:'+str(i[2]))
   graph.create(a)

for i in stock.values:
   a = Node('股票',TS代码=i[1],股票名称=i[3],行业=i[4])
   print('TS代码:'+str(i[1]),'股票名称:'+str(i[3]),'行业:'+str(i[4]))
   graph.create(a)

for i in holder.values:
   a = Node('股东',TS代码=i[0],股东名称=i[1],持股数量=i[2],持股比例=i[3])
   print('TS代码:'+str(i[0]),'股东名称:'+str(i[1]),'持股数量:'+str(i[2]))
   graph.create(a)
```

### 3.5 创建关系

股票-股东、股票-概念、股票-公告、股票-股通

```python
matcher = NodeMatcher(graph)
for i in holder.values:    
   a = matcher.match("股票",TS代码=i[0]).first()
   b = matcher.match("股东",TS代码=i[0])
   for j in b:
       r = Relationship(j,'参股',a)
       graph.create(r)
       print('TS',str(i[0]))
           
for i in concept.values:
   a = matcher.match("股票",TS代码=i[3]).first()
   b = matcher.match("概念",概念代码=i[1]).first()
   if a == None or b == None:
       continue
   r = Relationship(a,'概念属于',b)
   graph.create(r)

noticesdir = os.listdir("notices\\")
for n in noticesdir:
   notice = pd.read_csv("notices\\"+n,encoding="utf_8_sig")
   notice['content'] = notice['content'].fillna('空白')
   for i in notice.values:
       a = matcher.match("股票",TS代码=i[0]).first()
       b = Node('公告',日期=i[1],标题=i[2],内容=i[3])
       graph.create(b)
       r = Relationship(a,'发布公告',b)
       graph.create(r)
       print(str(i[0]))
       
for i in sz.values:
   a = matcher.match("股票",TS代码=i[0]).first()
   b = matcher.match("深股通").first()
   r = Relationship(a,'成分股属于',b)
   graph.create(r)
   print('TS代码:'+str(i[1]),'--深股通')

for i in sh.values:
   a = matcher.match("股票",TS代码=i[0]).first()
   b = matcher.match("沪股通").first()
   r = Relationship(a,'成分股属于',b)
   graph.create(r)
   print('TS代码:'+str(i[1]),'--沪股通')

# 构建股票间关联
corr = pd.read_csv("corr.csv")
for i in corr.values:
   a = matcher.match("股票",TS代码=i[1][:-1]).first()
   b = matcher.match("股票",TS代码=i[2][:-1]).first()
   r = Relationship(a,str(i[3]),b)
   graph.create(r)
   print(i)
```

## 4 数据可视化查询（以平安银行为例）

基于Crypher语言

### 4.1 查看关联实体

### 4.2 计算股票间对数收益率的相关系数后查看关联实体

### 4.3 查看平安银行与万科A之间的对数收益率的相关系数

## 5 neo4j 图算法目录、使用方法

### 5.1 目录

#### 5.1.1 中心度算法(Centralities) 

- PageRank (页面排名)

- ArticleRank

- Betweenness Centrality (中介中心度)

- Closeness Centrality (接近中心度)

- Harmonic Centrality

#### 5.1.2 社区检测算法(Community detection)   

- Louvain (鲁汶算法)

- Label Propagation (标签传播)

- Connected Components (连通组件)

- Strongly Connected Components (强连通组件)

Triangle Counting / Clustering Coefficient (三角计数/聚类系数)

#### 5.1.3 路径搜索算法(Path finding)  

- Minimum Weight Spanning Tree (最小权重生成树)

- Shortest Path (最短路径)

- Single Source Shortest Path (单源最短路径)

- All Pairs Shortest Path (全顶点对最短路径)

- A*

- Yen’s K-shortest paths

- Random Walk (随机漫步)

#### 5.1.4 相似性算法(Similarity)    

- Jaccard Similarity (Jaccard相似度)

- Cosine Similarity (余弦相似度)

- Pearson Similarity (Pearson相似度)

- Euclidean Distance (欧氏距离)

- Overlap Similarity (重叠相似度)

#### 5.1.5 链接预测(Link Prediction)

- Adamic Adar

- Common Neighbors

- Preferential Attachment

- Resource Allocation

- Same Community

- Total Neighbors

#### 5.1.6 预处理算法(Preprocessing)：

- One Hot Encoding

### 5.2 导入

（1）下载`graph-algorithms-algo-3.5.4.0.jar`复制到对应数据库的`plugin`文件夹下

（2）修改数据库目录下的`conf`中`neo4j.conf`，添加`dbms.security.procedures.unrestricted=algo.*`

### 5.3 eg.链路预测算法

实践neo4j的链路预测算法：主要基于判断相邻的两个节点之间的亲密程度作为评判标准

算法实践：

```cypher
MERGE (zhen:Person {name: "Zhen"})
MERGE (praveena:Person {name: "Praveena"})
MERGE (michael:Person {name: "Michael"})
MERGE (arya:Person {name: "Arya"})
MERGE (karin:Person {name: "Karin"})

MERGE (zhen)-[:FRIENDS]-(arya)
MERGE (zhen)-[:FRIENDS]-(praveena)
MERGE (praveena)-[:WORKS_WITH]-(karin)
MERGE (praveena)-[:FRIENDS]-(michael)
MERGE (michael)-[:WORKS_WITH]-(karin)
MERGE (arya)-[:FRIENDS]-(karin)

MATCH (p1:Person {name: 'Michael'})
MATCH (p2:Person {name: 'Karin'})
RETURN algo.linkprediction.adamicAdar(p1, p2) AS score
// score: 0.910349
                   
MATCH (p1:Person {name: 'Michael'})
MATCH (p2:Person {name: 'Karin'})
RETURN algo.linkprediction.adamicAdar(p1, p2, {relationshipQuery: "FRIENDS"}) AS score
// score: 0.0                                                
```

---

文本数据处理

获取当前财经新闻，并使用中文格式保存和查看

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/textdata.png">

使用wordcloud生成词云

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/wordcloud.jpg">

数据清理：过滤无关词语

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/filter2.png">

情感得分：

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/score.png">

#### 数据交互（Sample）

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/sample.png">

#### 数据存储（创建实体）

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/createEntity.png">

#### 数据存储（创建关系）

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/createRelation.png">

#### 数据可视化查询

查询与“平安银行”相关信息（所属概念板块、发布公告、属于深股通/沪股通、股东信息）

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match.png">

插入股票间相关系数之后，显示与“平安银行”所有相关信息

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match2.png">

查询“平安银行”与“万科A”的对数收益的相关系数

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match3.png">





![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict.png)

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict2.png)

#### 链路预测算法

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict3.png)

