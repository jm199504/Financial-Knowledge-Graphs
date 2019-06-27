## 小型金融知识图谱（Financial-Knowledge-Graphs）[基于Neo4j图数据库]<br>
##### 数据获取：data_download.ipynb<br>
##### 数据处理：data_download.ipynb<br>
##### 数据存储：data_storage.ipynb<br>
##### 流程细节：小型金融知识图谱构建示范.pdf<br>

##### 使用neo4j附带包：graph-algorithms-algo-3.5.4.0.jar<br>
##### （1）将graph-algorithms-algo-3.5.4.0.jar复制到对应数据库的plugin文件夹下
##### （2）修改数据库目录下的conf中neo4j.conf，添加dbms.security.procedures.unrestricted=algo.*
