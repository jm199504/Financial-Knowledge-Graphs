## Cypher Cheetsheet基础语法



### 1 创建节点

类型为`Person`（属性：姓名、年龄及性别）

```cql
create (:Person{name:"Tom",age:18,sex:"male"})
create (:Person{name:"Jimmy",age:20,sex:"male"})
```

### 2 创建关系

寻找2个Person类型节点分别姓名为Tom和Jimmy，创建两节点之间的关系：类型为Friend，关系值为best

```cql
match(p1:Person),(p2:Person)
where p1.name="Tom" and p2.name = "Jimmy"
create(p1) -[:Friend{relation:"best"}] ->(p2);
```

### 3 创建索引

```cql
create index on :Person(name)
// 创建唯一索引（属性值唯一）
create constraint on (n:Person) assert n.name is unique
```

### 4 删除节点

```cql
// 普通删除
match(p:Person_{name:"Jiimmy"}) delete p
match (a)-[r:knows]->(b) delete r,b
// 级联删除（即删除某个节点时会同时删除该节点的关系）
match (n{name: "Mary"}) detach delete n
// 删除所有节点
match (m) delete m
```

### 5 删除关系

```cql
// 普通删除
match(p1:Person)-[r:Friend]-(p2:Person)
where p1.name="Jimmy" and p2.name="Tom"
delete r
// 删除所有关系
match p=()-[]-() delete p
```

### 6 merge关键字

存在直接返回；不存在则新建并返回（通常实际用途于在对节点添加属性时避免报错）

```cql
// 创建/获取对象
merge (p:Person { name: "Jim1" }) return p;

// 创建/获取对象 + 设置属性值 + 返回属性值
merge (p:Person { name: "Koko" })
on create set p.time = timestamp()
return p.name, p.time

// 创建关系
match (a:Person {name: "Jim"}),(b:Person {name: "Tom"})
merge (a)-[r:friends]->(b)
```

### 7 更新节点

#### 7.1 更新属性值

```cql
match (n {name:'Jim'})
set n.name='Tom'
set n.age=20
return n
```

#### 7.2 新增属性和属性值

```cql
match (n {name:'Mary'}) set n += {age:20} return n
```

#### 7.3 删除属性值

```cql
match(n{name:'Tom'}) remove n.age return n
```

#### 7.4 更新节点类型（允许有多个标签）

```cql
①match (n{name:'Jim'}) set n:Person return n
②match (n{name:'Jim'}) set n:Person:Student return n
```

### 8 匹配

#### 8.1 限制节点类型和属性匹配

```cql
match (n:Person{name:"Jim"}) return n
match (n) where n.name = "Jim" return n
match (n:Person)-[:Realation]->(m:Person) where n.name = 'Mary'
```

#### 8.2 可选匹配（对于缺失部分使用Null代替）

```cql
optional match (n)-[r]->(m) return m
```

#### 8.3 字符串开头匹配

```cql
match (n) where n.name starts with 'J' return n
```

#### 8.4 字符串结尾匹配

```cql
match (n) where n.name ends with 'J' return n
```

#### 8.5 字符串包含匹配

```cql
match (n) where n.name contains with 'g' return n
```

#### 8.6 字符串排除匹配

```cql
match (n) where not n.name starts with 'J' return n
```

#### 8.7 正则匹配 =~（模糊匹配）

```cql
match (n) where n.name =~ '.*J.*' return n （等价） like '%J%'
```

#### 8.8 正则匹配 =~（不区分大小写）

```cql
match (n) where n.name =~ '(?i)b.*' return n （等价） like 'B/b%'
```

#### 8.9 属性值包含（IN）

```cql
match (n { name: 'Jim' }),(m) where m.name in ['Tom', 'Koo'] and (n)<--(m) return m
```

#### 8.10 "或"匹配（|）

```cql
match p=(n)-[:knows|:likes]->(m) return p
```

#### 8.11 任意节点和指定范围深度关系

```cql
match p=(n)-[*1..3]->(m) return p
```

#### 8.12 任意节点和指任意深度关系

```cql
match p=(n)-[*]->(m) return p
```

#### 8.13 去重返回

```cql
match (n) where n.ptype='book' return distinct n
```

#### 8.14 排序返回（desc降序；asc升序）

```cql
match (n) where n.ptype='book' return n order by n.price desc
```

#### 8.15 重命名返回

```cql
match (n) where n.ptype='book' return n.pname as name
```

#### 8.16 多重条件限制（with），即返回认识10人以上的张%

```cql
match (a)-[:knows]-(b)
where a.name =~ '张.*'
with a, count(b) as friends
where friends > 10
return a
```

#### 8.17 并集去重（union）

```cql
match (a)-[:knows]->(b) return b.name
union
match (a)-[:likes]->(b) eturn b.name
```

#### 8.18 并集不去重（union all）

```cql
match (a)-[:knows]->(b) return b.name
union all
match (a)-[:likes]->(b) eturn b.name
```

#### 8.19 查看节点属性/ID

```cql
match (p) where p.name = 'Jim' 
return keys(p)/properties(p)/id(p)
```

#### 8.20 匹配分页返回

```cql
match (n) where n.name='John' return n skip 10 limit 10
```



### 9 读取文件

#### 9.1 读取网络资源csv文件

```cql
load csv with header from 'url:[www.download.com/abc.csv](http://www.download.com/abc.csv)' as line

create (:Track{trackId:line.id,name:line.name,length:line.length})
```

#### 9.2 分批读取网络资源
例如 csv文件（default=1000）

```cql
using periodic commit (800)

load csv with header from 'url:[www.download.com/abc.csv](http://www.download.com/abc.csv)' as line

create (:Track{trackId:line.id,name:line.name,length:line.length})
```

#### 9.3 读取本地文件

```cql
load csv with headers from 'file:///00000.csv' as line
create (:Data{date:line['date'],open:line['open']})
(fieldterminator ';') //自定义分隔符
```

#### 9.4 注意事项

```
※ 本地csv文件必须是utf-8格式
※ 需要导入neo4j数据库目录的import目录下
※ 本地csv包含column必须添加with headers
```



### 10 foreach关键字



---

个人小结

1.节点属性使用`()`
2.关系属性使用`[]`
3.where中使用`"="`
4.`{}`中使用`":"`
5.关系建立使用`(m)-[:r]->(n)`
6.正则使用`"=~"`
7.节点或者关系(/[变量名:类型{属性名:属性值}]/)
8.匹配关系时需要基于p=(m)-[r]->(n)返回p，而不是返回r（显示空）