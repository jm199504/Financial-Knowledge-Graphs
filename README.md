**小型金融知识图谱搭建示范**

存储方式

1. 基于RDF的存储

2. 基于图数据库的存储

方式对比

**知识图谱构建流程**

1.数据获取

*股票基本信息   *股票Top10股东信息    *股票概念信息   *股票公告信息   
*财经新闻信息（该数据集已获取但需进一步处理，未存入图数据库）   *概念信息   *股票价格信息

2.数据预处理

*基本信息存在空值   *股东信息存在重复数据
*CSV文件格式更改为UTF-8格式    *计算股票对数收益   *保留股票价格交易日为242（众数）&计算皮尔逊相关系数

3.数据存储

*明确实体&关系    *使用py2neo交互neo4j创建节点和关系

4.数据可视化查询

*基于Crypher语言

5.相关应用

*中心度算法(Centralities)    *社区检测算法(Community detection)    *路径搜索算法(Path finding)   
*相似性算法(Similarity)    *链接预测(Link Prediction)

**数据获取**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/obtain.png)

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/obtain2png)

**数据预处理**

**数据交互（Sample）**

**数据存储（创建实体）**

**数据存储（创建关系）**

**数据可视化查询**

**链路预测算法**

使用neo4j附带的图算法，其中链路预测部分主要基于判断相邻的两个节点之间的亲密程度作为评判标准

**链路预测算法**

**其他算法**

