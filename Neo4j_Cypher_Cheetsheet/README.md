## Cypher Cheetsheet

**一、个人小结：**

1.节点属性使用()

2.关系属性使用[]

3.where中使用"="

4.{……}中使用":"

5.关系建立使用(m)-[:r]->(n)

6.正则使用"=~"

**二、基础语法**

**1.创建节点/关系**

**1.1节点(设置节点类型和属性和属性值)：**

create (:Person{name:"Tom",age:18,sex:"male"})

create (:Person{name:"Jimmy",age:20,sex:"male"})

**1.2创建关系（设置关系类型和属性和属性值）：**

match(p1:Person),(p2:Person) 

where p1.name="Tom" and p2.name = "Jim"

create(p1) -[:Relation{relation:"friend"] ->(p2);



**2.索引（在使用where查询时对索引限制时查询速度很快）**

**2.1创建索引**

create index on :Person(name)

**2.2删除索引**

drop index on :Person(name)

**2.3创建唯一索引（可以保证该属性值具有唯一性）**

create constraint on (n:Person) assert n.name is unique



**3.删除节点/关系：**

**3.1普通删除**

match(p:Person_{name:"Jiimmy"}) delete p

match (a)-[r:knows]->(b) delete r,b

**3.2级联删除（即删除某个节点时会同时删除该节点的关系）**

match (n{name: "Mary"}) detach delete n



**4.merge关键字（存在直接返回；不存在则新建并返回）：**

merge (p:Person { name: "Jim1" }) return p;

通常实际用途于在对节点添加属性时避免报错：

merge (p:Person { name: "Koko" })

on create set p.time = timestamp()

return p.name, p.time

在节点间添加关系：

match (a:Person {name: "Jim"}),(b:Person {name: "Tom"})
merge (a)-[r:friends]->(b)



**5.更新节点/关系：**

**5.1更新属性值**

match (n {name:'Jim'})

set n.name='Tom'

set n.age=20 

return n

**5.2新增属性和属性值**

match (n {name:'Mary'}) set n += {age:20} return n

**5.3删除属性值**

match(n{name:'Tom'}) remove n.age return n

**5.4更新节点类型/标签（允许有多个标签）**

①match (n{name:'Jim'}) set n:Person return n

②match (n{name:'Jim'}) set n:Person:Student return n



**6.匹配语句**

**6.1限制节点类型和属性匹配**

match (n:Person{name:"Jim"}) return n

match (n) where n.name = "Jim" return n

match (n:Person)-[:Realation]->(m:Person) where n.name = 'Mary'

**6.2可选匹配（对于缺失部分使用Null代替）**

optional match (n)-[r]->(m) return m

**6.3字符串开头匹配**

match (n)  where n.name starts with 'J'  return n

**6.4字符串结尾匹配**

match (n)  where n.name ends with 'J'  return n

**6.5字符串包含匹配**

match (n)  where n.name contains with 'g'  return n

**6.6字符串排除匹配**

match (n)  where not n.name starts with 'J'  return n

**6.7正则匹配 =~（模糊匹配）**

match (n)  where n.name =~ '.\*J.\*'  return n （等价） like '%J%'

**6.8正则匹配 =~（不区分大小写）**

match (n)  where n.name =~ '(?i)b.\*'  return n （等价） like 'B/b%'

**6.9属性值包含（IN）**

match (n { name: 'Jim' }),(m) where m.name in ['Tom', 'Koo'] and (n)<--(m)
return m

**6.10"或"匹配（|）**

match p=(n)-[:knows|:likes]->(m) return p

**6.11任意节点和指定范围深度关系**

match p=(n)-[\*1..3]->(m) return p

**6.12任意节点和指任意深度关系**

match p=(n)-[\*]->(m) return p

**6.13去重返回**

match (n) where n.ptype='book' return distinct n

**6.14排序返回（desc降序；asc升序）**

match (n) where n.ptype='book' return n order by n.price desc

**6.15重命名返回**

match (n) where n.ptype='book' return n.pname as name

**6.16多重条件限制（with），即返回认识10人以上的张%**

match (a)-[:knows]-(b)

where a.name =~ '张.\*'

with a, count(b) as friends

where friends > 10

return a

**6.17并集去重（union）**

match (a)-[:knows]->(b) return b.name

union

match (a)-[:likes]->(b) eturn b.name

**6.18并集不去重（union all）**

match (a)-[:knows]->(b) return b.name

union all

match (a)-[:likes]->(b) eturn b.name

**6.19查看节点属性/ID**

match (p) where p.name = 'Jim' return keys(p)/properties(p)/id(p)

**7.读取文件导入**

**7.1读取网络资源csv文件**

load csv with header from 'url:www.download.com/abc.csv' as line

create (:Track{trackId:line.id,name:line.name,length:line.length})

**7.2分批读取网络资源csv文件（default=1000）**

using periodic commit (800)

load csv with header from 'url:www.download.com/abc.csv' as line

create (:Track{trackId:line.id,name:line.name,length:line.length})

**7.3读取本地文件**

load csv with headers from 'file://00000.csv' as line

create (:Data{date:line['date'],open:line['open']})

(fieldterminator ';') //自定义分隔符

**※本地csv文件必须是utf-8格式**

**※需要导入neo4j数据库目录的import目录下**

**※本地csv包含column必须添加with headers**

