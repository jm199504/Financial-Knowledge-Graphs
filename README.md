## 小型金融知识图谱构流程示范

![](https://img.shields.io/static/v1?label=Author&message=jm199504&color=green)
![](https://img.shields.io/static/v1?label=Languages&message=python3.6+&color=orange)
![](https://img.shields.io/static/v1?label=LastUpdateTime&message=2020.09.22+&color=lightgrey)

### 知识图谱存储方式

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

### 关于neo4j安装、启动、登录

#### （1）下载

[下载链接](https://neo4j.com/download-center/)

#### （2）启动（neo4j目录）

```
cd neo4j/bin
./neo4j start
```

启动成功，终端会提示：

```
Starting Neo4j.Started neo4j (pid 30914). It is available at http://localhost:7474/ There may be a short delay until the server is ready.
```

#### a.打开 http://localhost:7474

#### b.初始账户和密码均为neo4j（host类型选择bolt）

#### c.输入旧密码并输入新密码

启动前注意本地已安装JDK（建议安装JDK版本11）：https://www.oracle.com/java/technologies/javase-downloads.html

完成安装JDK1.8.0_261后，在启动neo4j过程中出现了以下问题：

```
Unable to find any JVMs matching version "11"
```

解决：提示安装jdk 11 version，于是下载了jdk-11.0.8，Mac OS可通过`ls -la /Library/Java/JavaVirtualMachines/`查看已安装的jdk及版本信息。

#### （3）登录

---

### 知识图谱构建流程

#### 0.数据接口

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



#### 1.数据获取

（1）股票基本信息

```python
stock_basic = pro.stock_basic(list_status='L', fields='ts_code, symbol, name, industry')
# 重命名行，便于后面导入neo4j
basic_rename = {'ts_code': 'TS代码', 'symbol': '股票代码', 'name': '股票名称', 'industry': '行业'}
stock_basic.rename(columns=basic_rename, inplace=True)
# 保存为stock_basic.csv
stock_basic.to_csv('financial_data\\stock_basic.csv', encoding='gbk')
```



（2）股票持有股东信息    

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



（3）股票概念信息   

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



（4）股票公告信息   

```python
for i in range(3610):
   code = stock_basic['TS代码'].values[i]
   notices = pro.anns(ts_code=code, start_date='20180101', end_date='20181231', year='2018')
   notices.to_csv("financial_data\\notices\\"+str(code)+".csv",encoding='utf_8_sig',index=False)
notices = pro.anns(ts_code='000001.SZ', start_date='20180101', end_date='20181231', year='2018')
```



（5）财经新闻信息

```python
news = pro.news(src='sina', start_date='20180101', end_date='20181231')
news.to_csv("financial_data\\news.csv",encoding='utf_8_sig')
```



（6）概念信息   

```python
concept = pro.concept()
concept.to_csv('financial_data\\concept.csv', encoding='gbk')
```



（7）沪股通和深股通成分信息

```python
#获取沪股通成分
sh = pro.hs_const(hs_type='SH')
sh.to_csv("financial_data\\sh.csv",index=False)
#获取深股通成分
sz = pro.hs_const(hs_type='SZ')
sz.to_csv("financial_data\\sz.csv",index=False)
```



（8）股票价格信息

```python
for i in range(3610):
   code = stock_basic['TS代码'].values[i]
   price = pro.query('daily', ts_code=code, start_date='20180101', end_date='20181231')
   price.to_csv("financial_data\\price\\"+str(code)+".csv",index=False)
```



（9）其它类型数据（以`tushare`免费接口为例）

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



#### 2.数据预处理

(1)基本信息存在空值   

(2)股东信息存在重复数据

(3)CSV文件格式更改为UTF-8格式 

(4)计算股票对数收益   

(5)保留股票价格交易日为242（众数）&计算皮尔逊相关系数

#### 3.数据存储

(1)明确实体&关系    

(2)使用py2neo交互neo4j创建节点和关系

#### 4.数据可视化查询

(1)基于Crypher语言

#### 5.相关应用

(1)中心度算法(Centralities)    

(2)社区检测算法(Community detection)    

(3)路径搜索算法(Path finding)  

(4)相似性算法(Similarity)    

(5)链接预测(Link Prediction)

#### 数据获取

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/obtain.png">

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/obtain2.png">

#### 数据预处理

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/preprocess.png">

<img src="https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/preprocess2.png">

#### 文本数据处理

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

#### 导入已开源的图算法

（1）下载graph-algorithms-algo-3.5.4.0.jar复制到对应数据库的plugin文件夹下

（2）修改数据库目录下的conf中neo4j.conf，添加dbms.security.procedures.unrestricted=algo.*

#### 链路预测算法

使用neo4j附带的图算法，其中链路预测部分主要基于判断相邻的两个节点之间的亲密程度作为评判标准

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict.png)

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict2.png)

#### 链路预测算法

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict3.png)

#### 其他算法

##### 存储方式中心度算法(Centralities)：

（1）PageRank (页面排名)

（2）ArticleRank

（3）Betweenness Centrality (中介中心度)

（4）Closeness Centrality (接近中心度)

（5）Harmonic Centrality

##### 社区检测算法(Community detection)：

（1）Louvain (鲁汶算法)

（2）Label Propagation (标签传播)

（3）Connected Components (连通组件)

（4）Strongly Connected Components (强连通组件)

（5）Triangle Counting / Clustering Coefficient (三角计数/聚类系数)

##### 路径搜索算法(Path finding)：

（1）Minimum Weight Spanning Tree (最小权重生成树)

（2）Shortest Path (最短路径)

（3）Single Source Shortest Path (单源最短路径)

（4）All Pairs Shortest Path (全顶点对最短路径)

（5）A*

（6）Yen’s K-shortest paths

（7）Random Walk (随机漫步)

##### 相似性算法(Similarity)：

（1）Jaccard Similarity (Jaccard相似度)

（2）Cosine Similarity (余弦相似度)

（3）Pearson Similarity (Pearson相似度)

（4）Euclidean Distance (欧氏距离)

（5）Overlap Similarity (重叠相似度)

##### 链接预测(Link Prediction)：

（1）Adamic Adar

（2）Common Neighbors

（3）Preferential Attachment

（4）Resource Allocation

（5）Same Community

（6）Total Neighbors

##### 预处理算法(Preprocessing)：

（1）One Hot Encoding
