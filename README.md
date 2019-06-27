**小型金融知识图谱搭建示范**

存储方式

1. 基于RDF的存储

2. 基于图数据库的存储

方式对比

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/compare.png)

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

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/obtain2.png)

**数据预处理**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/preprocess.png)

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/preprocess2.png)

**数据交互（Sample）**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/sample.png)

**数据存储（创建实体）**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/createEntity.png)

**数据存储（创建关系）**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/createRelation.png)

**数据可视化查询**

查询与“平安银行”相关信息（所属概念板块、发布公告、属于深股通/沪股通、股东信息）

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match.png)

插入股票间相关系数之后，显示与“平安银行”所有相关信息

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match2.png)

查询“平安银行”与“万科A”的对数收益的相关系数

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/match3.png)

**链路预测算法**

使用neo4j附带的图算法，其中链路预测部分主要基于判断相邻的两个节点之间的亲密程度作为评判标准

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict.png)

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict2.png)

**链路预测算法**

![rank](https://github.com/jm199504/Financial-Knowledge-Graphs/blob/master/images/linkpredict3.png)

**其他算法**
中心度算法(Centralities)：

（1）PageRank (页面排名)

（2）ArticleRank

（3）Betweenness Centrality (中介中心度)

（4）Closeness Centrality (接近中心度)

（5）Harmonic Centrality

社区检测算法(Community detection)：

（1）Louvain (鲁汶算法)

（2）Label Propagation (标签传播)

（3）Connected Components (连通组件)

（4）Strongly Connected Components (强连通组件)

（5）Triangle Counting / Clustering Coefficient (三角计数/聚类系数)

路径搜索算法(Path finding)：

（1）Minimum Weight Spanning Tree (最小权重生成树)

（2）Shortest Path (最短路径)

（3）Single Source Shortest Path (单源最短路径)

（4）All Pairs Shortest Path (全顶点对最短路径)

（5）A*

（6）Yen’s K-shortest paths

（7）Random Walk (随机漫步)

相似性算法(Similarity)：

（1）Jaccard Similarity (Jaccard相似度)

（2）Cosine Similarity (余弦相似度)

（3）Pearson Similarity (Pearson相似度)

（4）Euclidean Distance (欧氏距离)

（5）Overlap Similarity (重叠相似度)

链接预测(Link Prediction)：

（1）Adamic Adar

（2）Common Neighbors

（3）Preferential Attachment

（4）Resource Allocation

（5）Same Community

（6）Total Neighbors

预处理算法(Preprocessing)：

（1）One Hot Encoding
