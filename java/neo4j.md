

# neo4j安装指导和Cypher语句





- [neo4j安装指导和Cypher语句](#neo4j安装指导和cypher语句)
    - [下载与安装](#下载与安装)
        - [1.Neo4j ubuntu安装教程](#1neo4j-ubuntu安装教程)
        - [1.创建节点](#1创建节点)
        - [2.查询节点](#2查询节点)
            - [2.1 查询整个图形数据库](#21-查询整个图形数据库)
            - [2.2 查询具有指定标签的节点](#22-查询具有指定标签的节点)
            - [2.3 where 谓词查询](#23-where-谓词查询)
        - [3.创建关系](#3创建关系)
            - [3.1 创建没有任何属性的关系](#31-创建没有任何属性的关系)
            - [3.2 创建关系，并设置关系的属性](#32-创建关系并设置关系的属性)
            - [3.3 创建双向关系](#33-创建双向关系)
        - [4 查询关系](#4-查询关系)
            - [4.1 查询整个数据图形](#41-查询整个数据图形)
            - [4.2 查询跟指定节点有关系的节点](#42-查询跟指定节点有关系的节点)
            - [4.3，查询有向关系的节点](#43查询有向关系的节点)
            - [4.4 为关系命名，通过\[r\]为关系定义一个变量名，通过函数type获取关系的类型](#44-为关系命名通过\r\为关系定义一个变量名通过函数type获取关系的类型)
            - [4.5 查询特定的关系类型，通过\[Variable:RelationshipType{Key:Value}\]指定关系的类型和属性](#45-查询特定的关系类型通过\variablerelationshiptypekeyvalue\指定关系的类型和属性)
        - [5 删除](#5-删除)
            - [5.1删除节点](#51删除节点)
            - [5.2 删除关系](#52-删除关系)
            - [5.3 强制删除节点和关系](#53-强制删除节点和关系)
        - [6 常用查询关键词](#6-常用查询关键词)
            - [6.1 count](#61-count)
            - [6.2 Limit](#62-limit)
            - [6.3 Distinct](#63-distinct)
            - [6.4 Order by](#64-order-by)
            - [6.5 根据id查找](#65-根据id查找)
            - [6.6 In的用法](#66-in的用法)
            - [6.7 Exists](#67-exists)
            - [6.8 With](#68-with)
            - [6.9 Contains](#69-contains)
            - [6.10 Union all (Union)](#610-union-all-union)
        - [7\. 更新](#7\-更新)
            - [7.1 创建一个完整的Path](#71-创建一个完整的path)
            - [7.2 为节点增加一个属性](#72-为节点增加一个属性)
            - [7.3 为节点增加标签](#73-为节点增加标签)
            - [7.4 为关系增加属性](#74-为关系增加属性)
            - [7.5 MERGE](#75-merge)
            - [7.6 跟实体相关的函数](#76-跟实体相关的函数)
        - [8.跟索引相关的函数](#8跟索引相关的函数)
            - [8.1创建索引](#81创建索引)
            - [8.2查询索引](#82查询索引)
            - [8.3删除索引](#83删除索引)
            - [8.4 备份与导入](#84-备份与导入)
        - [1.使用with关键字](#1使用with关键字)
        - [2.直接拼接关系节点查询](#2直接拼接关系节点查询)
        - [4.使用深度运算符](#4使用深度运算符)
        - [使用APOC库](#使用apoc库)
        - [[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)合并重复节点](#httpsneo4j-contribgithubioneo4j-apoc-procedureshttpsneo4j-contribgithubioneo4j-apoc-proceduresexport-importabrimg-src合并重复节点)
        - [安装apoc](#安装apoc)
        - [复制领域](#复制领域)
        - [keys函数](#keys函数)
        - [修改密码：](#修改密码)
        - [关于Neo4j和Cypher批量更新和批量插入优化的5个建议](#关于neo4j和cypher批量更新和批量插入优化的5个建议)
        - [Neo4j之Cypher学习总结](#neo4j之cypher学习总结)

## 下载与安装

### 1.Neo4j ubuntu安装教程

**首先使用**：
Debian repository:

```bash
     wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add - echo 'deb https://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list

    sudo apt-get update
```

**之后选择社区版安装**：

```bash
    sudo apt-get install neo4j
```

安装可能比较慢，别急

**然后配置基本信息**：

```bash
      cd /etc/neo4j
      sudo gedit neo4j.conf
```

进入文档将50行前面的注释去掉
dbms.connector.bolt.listen_address=0.0.0.0:7687

进入文档将第54行前面的注释去掉
dbms.connectors.default_listen_address=0.0.0.0：7474

保存之后，便可以重启
重启命令：neo4j restart
不报错就可以直接使用了

**如果此时报错**，没有这个文件或者文件夹

那就执行以下操作

```bash
mkdir /var/run/neo4j
neo4j start
```

之后在浏览器上访问
<服务器ip地址>:7474/browser/即可看到，初始账户和密码为neo4j，之后自己修改

Active database: graph.db  
Directories in use:  
  home:         /var/lib/neo4j  
  config:       /etc/neo4j  
  logs:         /var/log/neo4j  
  plugins:      /var/lib/neo4j/plugins  
  import:       /var/lib/neo4j/import  
  data:         /var/lib/neo4j/data  
  certificates: /var/lib/neo4j/certificates  
  run:          /var/run/neo4j  
Starting Neo4j.

### 1.创建节点

创建Person 标签,刘德华等若干节点,各自有name,birthday ,born,englishname等属性

1. `match(a),(b) where a.uuid=5110 and b.uuid=12110 create (a)-[r:父亲 {name:"父亲",properties:{relationID:521,time:"1997-05-12"},relationID:521}]->(b)`
2. `create (n:Person { name: '朱丽倩', birthday:'1966年4月6日',born: 1966 ,englishname:'Carol'}) ;`
3. `create (n:Person { name: '刘向蕙', birthday:'2012年5月9日',born: 2012 ,englishname:'Hanna'}) ;`
4. `create (n:Person { name: '任贤齐', birthday:'1966年6月23日',born: 1966 ,englishname:'Richie Jen'}) ;`
5. `create (n:Person { name: '金城武', birthday:'1973年10月11日',born: 1973,englishname:'Takeshi Kaneshiro'}) ;`
6. `create (n:Person { name: '林志玲', birthday:'1974年11月29日',born: 1974,englishname:'zhilin'}) ;`
7. `创建Movie 标签,彩云曲等若干节点,各自有title,released 等属性`
8. `create (n:Movie { title: '彩云曲',released: 1981})`
9. `create (n:Movie { title: '神雕侠侣',released: 1983})` 
10. `create (n:Movie { title: '暗战',released: 2000})`
11. `create (n:Movie { title: '拆弹专家',released: 2017})`

### 2.查询节点

#### 2.1 查询整个图形数据库

点击节点，查看节点的属性，如图，Neo4j自动为节点设置ID值  
`match(n) return n;`

![](http://file.miaoleyan.com/nndt/RTLNZTpcpXFmXkqEFgghgOSCquXrX6Lb)

#### 2.2 查询具有指定标签的节点

查询Movie标签下的节点  
`match(n:Movie) return n;`  
![](http://file.miaoleyan.com/nndt/ONuJJJ9L6T3qUoaJ0OBNf2GzB7lELvsr)

#### 2.3 where 谓词查询

查询名称为林志玲的节点  
`match (n:Person) where n.name='林志玲' return n`  
![](http://file.miaoleyan.com/nndt/X7DxApWede4g8tTxWKMYcQQmoOsLQOLt)
(查询具有指定属性的节点)结果相同  
`match(n{name:'林志玲'}) return n;`  
![](http://file.miaoleyan.com/nndt/OPQJDR5m291S7ERXmzSAdFVFRIYclSuL)
查询born属性小于1967的节点  
`match(n) where n.born<1967 return n;`

### 3.创建关系

关系的构成：StartNode - \[Variable:RelationshipType{Key1:Value1,Key2:Value2}\] -> EndNode，在创建关系时，必须指定关系类型。

#### 3.1 创建没有任何属性的关系

1. `MATCH (a:Person),(b:Movie)`
2. `WHERE a.name = '刘德华' AND b.title = '暗战'`
3. `CREATE (a)-[r:DIRECTED]->(b)`
4. `RETURN r;`

![](http://file.miaoleyan.com/nndt/Fqjr5iaDpgDOQg7LduUKIhFzaysqPHtZ)

#### 3.2 创建关系，并设置关系的属性

1. `MATCH (a:Person),(b:Movie)`
2. `WHERE a.name = '刘德华' AND b.title = '神雕侠侣'`
3. `CREATE (a)-[r:出演 { roles:['杨过'] }]->(b)`
4. `RETURN r;`

![](http://file.miaoleyan.com/nndt/NV8VSDingSlUjsLnjL3THI37R3CuukLx)

#### 3.3 创建双向关系

刘德华的女是刘向蕙,刘向蕙的父亲是刘德华

1. `MATCH (a:Person),(c:Person)`
2. `WHERE a.name = '刘德华' AND c.name = '刘向蕙'`
3. `CREATE (a)-[r:父亲 { nickname:'甜心' }]->(c),`
4. `(c)-[d:女儿 { nickname:'爹地' }]->(a)`
5. `RETURN r;`

![](http://file.miaoleyan.com/nndt/KwSusLqS5USIc9ZtT81Xo1A8hs9IqEiS)  
关系建错了 删除关系 (见5.2 )  
重新创建

1.  `MATCH (a:Person),(c:Person)`
2.  `WHERE a.name = '刘德华' AND c.name = '刘向蕙'`
3.  `CREATE (a)-[d:女儿 { nickname:'甜心' }]->(c),`
4.  `(c)-[r:父亲 { nickname:'爹地' }]->(a)`
5.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/dPKmVIkfEp88kf0IPqvlkhxzn0Ifu2aX)

1.  `MATCH (a:Person),(c:Movie)`
2.  `WHERE a.name = '刘德华' AND c.title = '彩云曲'`
3.  `CREATE (a)-[r:出演 { partner:'张国荣' }]->(c),`
4.  `(c)-[d:演员 { rolename:'阿华哥' }]->(a)`
5.  `RETURN r;`
6.  `MATCH (a:Person),(c:Movie)`
7.  `WHERE a.name = '刘德华' AND c.title = '拆弹专家'`
8.  `CREATE (a)-[r:出演 { partner:'赵薇,高圆圆' }]->(c),`
9.  `(c)-[d:演员 { rolename:'华仔' }]->(a)`
10.  `RETURN r;`
11.  `MATCH (a:Person),(c:Movie)`
12.  `WHERE a.name = '刘德华' AND c.title = '神雕侠侣'`
13.  `CREATE (a)-[r:出演 { partner:'刘亦菲' }]->(c),`
14.  `(c)-[d:演员 { rolename:'杨过' }]->(a)`
15.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/tRgmHlikkn2GYfIiZF2FrKgpEbxlnrWN)

1.  `继续新增关系 刘德华和林志玲,金城武,任贤齐`
2.  `MATCH (a:Person),(c:Person)`
3.  `WHERE a.name = '刘德华' AND c.name = '任贤齐'`
4.  `CREATE (a)-[d:朋友 { sex:'男' }]->(c)`
5.  `RETURN d;`

7.  `MATCH (a:Person),(c:Person)`
8.  `WHERE a.name = '刘德华' AND c.name = '金城武'`
9.  `CREATE (a)-[d:朋友 { sex:'男' }]->(c)`
10.  `RETURN d;`
11.   `这里没有给关系设置属性sex`
12.  `MATCH (a:Person),(c:Person)`
13.  `WHERE a.name = '刘德华' AND c.name = '林志玲'`
14.  `CREATE (a)-[d:朋友]->(c)`
15.  `RETURN d;` 

17.  `查询Person表关系`
18.  `MATCH (n:Person) RETURN n`

![](http://file.miaoleyan.com/nndt/pBQJwvrnzIAN1eUV6oNFZMutUjl0eHhH)

### 4 查询关系

在Cypher中，关系分为三种：符号“—”，表示有关系，忽略关系的类型和方向；符号“—>”和“<—”，表示有方向的关系；

#### 4.1 查询整个数据图形

`match(n) return n;`

![](http://file.miaoleyan.com/nndt/8Gy521m3erXIR320B7X2KYp73VQ1cKvn)

#### 4.2 查询跟指定节点有关系的节点

查询跟Movie标签有关系的所有节点

![](http://file.miaoleyan.com/nndt/W9gtIB9wA7BxJGGWhsfT4pQvZpdpmjbK)

#### 4.3，查询有向关系的节点

查询和刘德华有关系的电影  
`MATCH (:Person { name: '刘德华' })-->(movie)RETURN movie;`

![](http://file.miaoleyan.com/nndt/xbfKJHuNPq9oZ46rwNSbYKQtAuyLV874)

#### 4.4 为关系命名，通过\[r\]为关系定义一个变量名，通过函数type获取关系的类型

1.  `MATCH (:Person { name: '刘德华' })-[r]->(movie) RETURN r,type(r);`

![](http://file.miaoleyan.com/nndt/nrhktZohsHEQhoMdOcAwV6Fh10As7Z30)

#### 4.5 查询特定的关系类型，通过\[Variable:RelationshipType{Key:Value}\]指定关系的类型和属性

`MATCH (:Person { name: '刘德华' })-[r:出演{partner:'张国荣'}]->(Movie) RETURN r,type(r);`

![](http://file.miaoleyan.com/nndt/Nlc3Fii8HQGq4LTaM3870TyCUICxnGcu)  
查询和刘德华和张国荣合作过的电影  
`MATCH (:Person { name: '刘德华' })-[r:出演{partner:'张国荣'}]->(m:Movie) RETURN m;`

![](http://file.miaoleyan.com/nndt/wfEwwVuu2FenkA5EePSBls6byYHW3c9m)  
查询被刘德华称呼为甜心的女儿  
`MATCH (:Person { name: '刘德华' })-[r:女儿{nickname:'甜心'}]->(m:Person) return m`  
查询刘德华的老婆是谁  
`Match (n:Person{name: '刘德华'})-[:wife]->(a:Person) return a`

![](http://file.miaoleyan.com/nndt/2Cjh7lcfJGmM1RZCwcj1BlFSpyVaDmlX)  
刘德华出演过的电影

![](http://file.miaoleyan.com/nndt/jXg8WZcEBwqggczfGeO9eqzfAqcBgrFw)

### 5 删除

#### 5.1删除节点

1.  `create (n:City { name: '北京'})`
2.  `Match (n:City{name:'北京'}) delete n`

#### 5.2 删除关系

1.  `Match (a:Person{name:'刘德华'})-[r:父亲]->(b:Person{name:'刘向蕙'}) delete r`
2.  `Match (a:Person{name:'刘向蕙'})-[r:女儿]->(b:Person{name:'刘德华'}) delete r`

#### 5.3 强制删除节点和关系

1.  ``Match (n:`美国军事基地`) where n.name ='挂载' detach delete n``

### 6 常用查询关键词

#### 6.1 count

查询Person 一共有多少人  
`Match (n:Person ) return count(n)`

![](http://file.miaoleyan.com/nndt/zIhhZ6gGOJn6Hc53fY9Ighs2OMS35yvR)  
查询标签(Person)中born=1966的一共有多少节点（人）：  
三种写法(第三种不能用似乎)：  
`1. Match (n:Person) where n.born=1966 return count(n)`

![](http://file.miaoleyan.com/nndt/AML1BD0lwnn1JDatuIAUwrW2kxDtRT9i)

1.  Match (n:Person{born:1966}) return count(n)//特别注意类型,如果存的类似是数字类型,使用字符串就查不出来,如下:

![](http://file.miaoleyan.com/nndt/4iF1zmuD74Q4YOHN1NbmOYmZ1p4gEtex)

![](http://file.miaoleyan.com/nndt/sGlRc1ZF1Op3JOe9ZfArHfh5iSuxHNNY)

#### 6.2 Limit

`Match (n:Person) return n limit 3`

![](http://file.miaoleyan.com/nndt/rHVr6899KoS5TKkXprShyib7AYo4YFYS)

![](http://file.miaoleyan.com/nndt/nbc9h3b1wsWzE4dvAGqokWNlOvqqwmSJ)

#### 6.3 Distinct

`Match (n:Person) return distinct(n.born)`

![](http://file.miaoleyan.com/nndt/Q3SWdrVwcPEX9FoFdtFU5Cti9GwZ6tn2)

#### 6.4 Order by

1.  `Match(n:Person) return n  order by n.born (默认升序)`
2.  `Match(n:Person) return n  order by n.born  asc (升序)`
3.  `Match(n:Person) return n  order by n.born  desc (降序)`

![](http://file.miaoleyan.com/nndt/j7yfWb7E5WiQQMqanHN0QhROengjMy3o)

![](http://file.miaoleyan.com/nndt/Ulj7j0vsdA3li6jFDQSQYF4bB7wINi0L)

![](http://file.miaoleyan.com/nndt/zBZgSDBc6lGh0Qhp1RiUYcPUHJSMrWBV)

#### 6.5 根据id查找

`match (n) where id(n)=548 return n`

![](http://file.miaoleyan.com/nndt/jN65PxJkcKdLZMHBCnH9d8dIVoxyqqNb)

#### 6.6 In的用法

1.  `Match (n) where ID(n) IN[353,145,547] return n` 
2.  `Match (n) where ID(n) IN[145,175,353,547,548] return n`

![](http://file.miaoleyan.com/nndt/tburFLmcTk8PVUbKgGesQc4tCktk4LIe)

#### 6.7 Exists

节点存在 name这个属性的记录：  
`Match (n) where exists(n.title) return n`

![](http://file.miaoleyan.com/nndt/uOyPBc2F2iQBG3rloKiCRlBPqpXAIvSo)

#### 6.8 With

查询name以‘刘’开头的节点：  
`Match (n) where n.name starts with '刘' return n`

![](http://file.miaoleyan.com/nndt/pbadSnGVkvqa5ZVHgLT3ALVcE5MICy6m)

#### 6.9 Contains

查询title中含有 ‘侠侣’的节点  
`Match (n:Movie) where n.title Contains '侠侣' return n`

![](http://file.miaoleyan.com/nndt/06SgibFRfiOkCIWotMNJ7y8GSRmsBCKZ)

#### 6.10 Union all (Union)

求并集，不去重（去重用Union, as 取别名）:

1.  `Match(n:Person) where n.born=1966 return n.name as name`
2.  `Union all`
3.  `Match(n:Movie) where n.released=1983 return n.title as name`

![](http://file.miaoleyan.com/nndt/Hl1koSGFVR9S3cCQfeAIz016YDXo7JQc)

### 7\. 更新

#### 7.1 创建一个完整的Path

1.  `CREATE p =(m:Person{ name:'刘亦菲',title:"演员" })-[:签约]->(neo)<-[:签约]-(n:Person { name: '赵薇',title:"投资人" })`
2.  `RETURN p`

![](http://file.miaoleyan.com/nndt/3ZsdZBQ0oLWracGYir7T1r66zUZ3unWX)

![](http://file.miaoleyan.com/nndt/r4vd7YLQb6C8pP5JS7XEHymLvAQ40QOE)

#### 7.2 为节点增加一个属性

通过节点的ID获取节点，Neo4j推荐通过where子句和ID函数来实现。

1.  `match (n)`
2.  `where id(n)=358`
3.  `set n.name = '华谊兄弟'`
4.  `return n;`

为节点移除属性  
`MATCH (n:农业) where id(n)=17816137 remove n.sortcode,n.targettable,n.unit,n.uuid`

![](http://file.miaoleyan.com/nndt/n9c6OehJBPF6outqWhx2ZBY73XKIt0ZC)

#### 7.3 为节点增加标签

1.  `match (n)`
2.  `where id(n)=358`
3.  `set n:公司`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/JKUqi66mJCfU500VOEKYEG3YDOTpZ1bS)

#### 7.4 为关系增加属性

1.  `match (n)-[r]->(m)`
2.  `where id(n)=357 and id(m)=358`
3.  `set r.经纪人='程晨'`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/wKCcFjUsfqNwD54OwqD4nPHXuROlw11F)  
此时图谱效果成

![](http://file.miaoleyan.com/nndt/I3CPxRfHSDEGvoJubB9Abf5WYiN9hhhI)  
接着让刘德华也和华谊兄弟签约

1.  `MATCH (a:Person),(c:公司)`
2.  `WHERE a.name = '刘德华' AND c.name = '华谊兄弟'`
3.  `CREATE (a)-[d:签约 { 经纪人:'刘得得' }]->(c)`
4.  `RETURN d;`

![](http://file.miaoleyan.com/nndt/FvG0DsSsHtRyFCgHVT3118ILcNeFvMPW)

#### 7.5 MERGE

Merge子句的作用有两个：当模式（Pattern）存在时，匹配该模式；当模式不存在时，创建新的模式，功能是match子句和create的组合。在merge子句之后，可以显式指定on creae和on match子句，用于修改绑定的节点或关系的属性。  
通过merge子句，你可以指定图形中必须存在一个节点，该节点必须具有特定的标签，属性等，如果不存在，那么merge子句将创建相应的节点。  
通过merge子句匹配搜索模式  
匹配模式是：一个节点有Person标签，并且具有name属性；如果数据库不存在该模式，那么创建新的节点；如果存在该模式，那么绑定该节点；

1.  `MERGE (m:Person { name: '迈克尔·杰克逊' }) RETURN m;`

![](http://file.miaoleyan.com/nndt/CmWHpmopMlMCRvMpv2FFcD1GrXWeeuAM)  
在merge子句中指定on create子句  
如果需要创建节点，那么执行on create子句，修改节点的属性；

1.  `MERGE (m:Person { name: '杰森·斯坦森' })`
2.  `ON CREATE SET m.registertime = timestamp()`
3.  `RETURN m.name, m.registertime`

![](http://file.miaoleyan.com/nndt/bi4rNu0KGVPuwgkP6EpOTYjUT8srcuMD)  
在merge子句中指定on match子句  
如果节点已经存在于数据库中，那么执行on match子句，修改节点的属性；节点属性不存在则新增

1.  `MERGE (m:Person)`
2.  `ON MATCH SET m.registertime = timestamp()`
3.  `RETURN m.name, m.registertime`

![](http://file.miaoleyan.com/nndt/wNflMULIU6uB4xLBrfJpiVMzwirf0Lf5)  
在merge子句中同时指定on create 和 on match子句(没有对应属性则修改不成功,不会新增属性)

1.  `MERGE (m:Person{ name: '李连杰' })`
2.  `ON CREATE  SET m.registertime = timestamp()`
3.  `ON MATCH SET m.offtime = timestamp()`
4.  `RETURN m.name, m.registertime,m.offtime`

![](http://file.miaoleyan.com/nndt/3XD3diqzwUsFiu1zszZNkkKWWEQ0eWmV)  
merge子句用于match或create一个关系

1.  `MATCH (m:Person { name: '刘德华' }),(n:Movie { title: '神雕侠侣' })`
2.  `MERGE (m)-[r:导演]->(n)`
3.  `RETURN m.name, type(r), n.title`

![](http://file.miaoleyan.com/nndt/GJFoWbQaleDaQGS45AK2U6UWfHVAUOQg)  
merge子句用于match或create多个关系  
赵薇既是神雕侠侣的导演,也是神雕侠侣的演员

1.  `MATCH (m:Person { name: '赵薇' }),(n:Movie { title: '神雕侠侣' })`
2.  `MERGE (m)-[r:导演]->(n)<-[r2:出演]-(m)`
3.  `RETURN m.name, type(r),type(r2), n.title`

![](http://file.miaoleyan.com/nndt/b2cxKGhpGRhdstnPb3W4I5lpezii9Zok)  
merge子句用于子查询  
先添加基础数据  
创建城市节点

1.  `create (n:City { name: '北京',othername:'帝都'})`
2.  `create (n:City { name: '香港',othername:'HongKong'})`
3.  `create (n:City { name: '台湾',othername:'湾湾'})`

为Person标签的每个节点都增加一个属性 bornin

1.  `match (n:Person)`
2.  `set n.bornin = ''`
3.  `return n;`

![](http://file.miaoleyan.com/nndt/Fbkg8VXyxHwQj4pWG9VGVjv7cw4sXx46)  
match (n)  
where id(n)=175  
set n.bornin = ‘香港’  
return n;

![](http://file.miaoleyan.com/nndt/US8xuWIAsjlshqekJiLaAt4bnYKCkfNh)

1.  `match (n)`
2.  `where n.name='金城武'`
3.  `set n.bornin = '台湾'`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/32KRi3ietIMC8mHwOz7B6zRy1wM2qZbF)  
需求:查找刘德华和金城武的信息和所在地的othername(相当于mysql 连表查询)

1.  `MATCH (p:Person)`
2.  `where p.name='刘德华' or p.name='金城武'`
3.  `MERGE (c:City { name: p.bornin })`
4.  `RETURN p.name,p.born,p.bornin , c.othername;`

![](http://file.miaoleyan.com/nndt/VzAbRCB535csGqMSWbYbZyRkYtxs5HVd)  
创建刘德华出生地是香港这条关系

1.  `MATCH (a:Person),(c:City)`
2.  `WHERE a.name = '刘德华' AND c.name = '香港'`
3.  `CREATE (a)-[r:出生地]->(c)`
4.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/Epmlnzzllew51X7XaIJs4tyyRlZcDanR)  
需求:给Person中每个节点都创建一个出生地的关系,没有则返回null

1.  `MATCH (p:Person)`
2.  `MERGE (c:City { name: p.bornin })`
3.  `MERGE (p)-[r:出生地]->(c)`
4.  `RETURN p.name, p.bornin, c.othername;`

![](http://file.miaoleyan.com/nndt/7boDFnBp6goe9ZKuEHQ9GvCsbdZN1dQi)

![](http://file.miaoleyan.com/nndt/AN7zmSwWDXkFps1rbFjqERwrzaF6zwfZ)  
删除这些关系

1.  `Match (a:Person)-[r:出生地]->(c:City{name:''}) delete r`
2.  `Match (a:City)-[r:出生地]->(c:Person) delete r`

![](http://file.miaoleyan.com/nndt/yh2ule0aJWUqct24hYhJBkiwUqQAglkm)  
查询Person标签中不存在name属性的节点

1.  `Match (n:Person) where not exists(n.name) return n`
2.  `Match (n:Person) where not exists(n.name) delete n`

![](http://file.miaoleyan.com/nndt/WnvCH8SZSXuT7DJJtxeasCWH7XDVDagB)  
create (n:Prize { name: ‘金马奖’});  
create (n:Prize { name: ‘奥斯卡金奖’});  
create (n:Prize { name: ‘金鸡奖’});  
create (n:Prize { name: ‘香港电影金像奖’});

#### 7.6 跟实体相关的函数

通过id函数，返回节点或关系的ID  
查询Person标签中和刘德华有关系的 id(节点和关系)

`MATCH (:Person { name: '刘德华' })-[r]->(movie) RETURN id(r);`

![](http://file.miaoleyan.com/nndt/H5LdHRTguYNVlMwMQsNVegVNn9gnyNZN)

通过type函数，查询关系的类型  
查询Person标签中和刘德华相关的关系(以下三种结果相同)

1.  `MATCH (:Person { name: '刘德华' })-[r]->(a)`
2.  `MATCH (:Person { name: '刘德华' })-[r]->(b)`
3.  `MATCH (:Person { name: '刘德华' })-[r]->()`
4.  `RETURN type(r);`

![](http://file.miaoleyan.com/nndt/SCfmXdAKG5E4R3aYUozB77HDoZnJF7BS)

![](http://file.miaoleyan.com/nndt/M6pWHBJpoVCjIWBOCrkHeBmwo0iOX8k6)  
通过lables函数，查询节点的标签  
查询和刘德华有关系的节点

1.  `MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN p;`

![](http://file.miaoleyan.com/nndt/HRw9QgLKVedhryFsJARXaERZh1cdrrlx)  
查询和刘德华有关系的标签(去重)

1.  `MATCH (:Person { name: '刘德华' })-[r]->(p)`
2.  `RETURN distinct(labels(p))`

![](http://file.miaoleyan.com/nndt/PCxlOZAJY6MEvfhjxLLcBEBucWUvusDS)  
通过keys函数，查看节点或关系的属性键

1.  `MATCH (a)`
2.  `WHERE a.name = '刘德华'`
3.  `RETURN keys(a)`

![](http://file.miaoleyan.com/nndt/wGYxL2fe4tIGMRSyU5JVuZXcN8KU84yc)  
`MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN distinct(keys(r))`

![](http://file.miaoleyan.com/nndt/NTFr6cZ2lFwpmZKk8tWtQaGdjSlolo3J)  
通过properties()函数，查看节点或关系的属性

1.  `MATCH (a)`
2.  `WHERE a.name = '刘德华'`
3.  `RETURN properties(a)`

![](http://file.miaoleyan.com/nndt/BL6DS5eJSpB13XGATi0nJ4UZmyIl5wyn)  
`MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN properties(r)`

![](http://file.miaoleyan.com/nndt/4yPRBh1PGUqJpHVvC2QHPTcdYE0w6emp)

### 8.跟索引相关的函数

#### 8.1创建索引

创建单一属性索引

1.  `CREATE INDEX ON :Lable(property)`
2.  `Query:`
3.  `CREATE INDEX ON :Person(name)`
4.  `给数据库的:Person标签的name属性创建索引`
5.  `Result:`
6.  `No data returned, and nothing was changed`
7.  `创建复合属性索引`
8.  `CREATE INDEX ON :Label(prop1,...,propN)`
9.  `Query:`
10.  `CREATE INDEX ON :Person(age,country)`
11.  `Result:`
12.  `No data returned.`
13.  `Indexes added:1`

#### 8.2查询索引

CALL db.Indexes

![](http://file.miaoleyan.com/nndt/rCE5mke8jIqPrM5aO8WnHXNCbGVeXUQT)

#### 8.3删除索引

1.  `DROP INDEX ON :LABEL(property）`
2.  `Query:`
3.  `DROP INDEX ON :Person(firstname)`
4.  `Result:`
5.  `No data returned`
6.  `Indexes removed:1`

8.  `DROP INDEX ON :LABEL(prop1,...,propN)`
9.  `Query:`
10.  `DROP INDEX ON :Person(age,country)`
11.  `Result:`
12.  `No data returned`
13.  `Indexes removed:1`

#### 8.4 备份与导入

备份/导出  
要以管理员身份运行  
`neo4j-admin dump --database=graph.db --to=E:\neo4jdata`

![](http://file.miaoleyan.com/nndt/BGvBvFpGCMm5Nn6TlC4dKTGcWmE6vl5L)

![](http://file.miaoleyan.com/nndt/kct9pwEo1HYeZYnWwfwcS0EA3a4yxJFH)  
导入  
`neo4j-admin load --from=E:\neo4jdata\graph.db.dump --database=graph.db --force`

导入csv

1.  ``USING PERIODIC COMMIT 500 LOAD CSV FROM "file:///D:\\test.csv" AS line  MERGE (:`顶顶顶` {name:line[0]})``
2.  `LOAD CSV WITH HEADERS FROM "file:///C:\\Program Files\\Java\\neo4j-community-3.4.7\\import\\stock_concept.csv"`
3.  `AS line`
4.  `return line.n`

![](http://file.miaoleyan.com/nndt/WuOaUz2UvKWkCKAa9c5bz148YZQ7Q53d)  
USING PERIODIC COMMIT 10  
LOAD CSV FROM “file:///node.csv” AS line  
create (a:Node{name:line\[0\]})

![](http://file.miaoleyan.com/nndt/waTKUTyuMYliPrutDWbGhR0hsyTRcOJ3)  
USING PERIODIC COMMIT  
LOAD CSV FROM ‘file:///concept.csv’ AS row  
CREATE (n:概念{name:row\[1\],uuid:row\[0\]})

csv 不带header方式  
USING PERIODIC COMMIT  
LOAD CSV FROM “file:///executive\_stock.csv” AS row  
MATCH (n:高管 {uuid: row\[0\]})  
MATCH (m:企业 {uuid: row\[1\]})  
MERGE (n)-\[:RE{name:row\[2\]}\]->(m)

带header方式  
USING PERIODIC COMMIT  
LOAD CSV WITH HEADERS FROM ‘file:///industry.csv’ AS row  
CREATE (:行业{name:row.name,uuid:row.uuid})

关于neo4j查询多深度关系节点
----------------

### 1.使用with关键字

1.   `查询三层级关系节点如下：with可以将前面查询结果作为后面查询条件`
2.  `match (na:company)-[re]->(nb:company) where na.id = '12399145' WITH na,re,nb match (nb:company)-[re2]->(nc:company) return na,re,nb,re2,nc`

### 2.直接拼接关系节点查询

`match (na:company{id:'12399145'})-[re]->(nb:company)-[re2]->(nc:company) return na,re,nb,re2,nc`

3.为了方便，可以将查询结果赋给变量，然后返回
-----------------------

`match data=(na:company{id:'12399145'})-[re]->(nb:company)-[re2]->(nc:company) return data`

### 4.使用深度运算符

当实现多深度关系节点查询时，显然使用以上方式比较繁琐。  
可变数量的关系->节点可以使用-\[:TYPE_minHops..maxHops\]->。  
查询：  
如果在1到3的关系中存在路径，将返回开始点和结束点。  
\`match data=(na:company{id:’12399145’})-\[_1..3\]->(nb:company) return data\`

### 使用APOC库

[https://neo4j-contrib.github.io/neo4j-apoc-procedures/](https://neo4j-contrib.github.io/neo4j-apoc-procedures/)  
导入导出的几种方式  
[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)

[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)

### [](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)合并重复节点

### 安装apoc

过程如下  
[https://blog.csdn.net/graphway/article/details/78957415](https://blog.csdn.net/graphway/article/details/78957415)  
安装完成 执行 return apoc.version() 查看下版本  
![](http://file.miaoleyan.com/nndt/ipPR41bCgMDDeT92Jcsu2KGQU4dFq0BB)  
MATCH (n:国家电网)  
WITH n.name AS name, COLLECT(n) AS nodelist, COUNT(\*) AS count  
WHERE count > 1  
CALL apoc.refactor.mergeNodes(nodelist) YIELD node  
RETURN node

查询某个节点有关系的3级及以内的路径  
MATCH (n:`贵州`) WHERE n.name=’交通事件’  
CALL apoc.path.spanningTree(n, {maxLevel:3}) YIELD path  
RETURN path;

### 复制领域

1.  `match(n:菊花) MERGE (:大萨达{name:n.name})`
2.  `match(n:菊花)-[r]->(q:菊花)`
3.  `with n, r, q`
4.  `match (o:大萨达{name:n.name}), (m:大萨达{name:q.name})`
5.  `MERGE (o)-[:RE{name:r.name}]->(m)`

### keys函数

1.  `match(n) where any(x in keys(n) where n[x] > 0) return n 查询某个属性大于0 的节点`
2.  `match(n) where all(x.uuid in keys(n) where n[x.uuid] > 0) return n 查询所有属性大于0的节点`

例如:x在any中是一个变量，并不是属性  
match(n) where any(uuid in keys(n) where n\[uuid\] > 0) return n uuid大于0  
match(n) where all(uuid in keys(n) where n\[uuid\] > 0) return n 所有uuid都大于0  
match(n) where any(querytype in keys(n) where n\[querytype\] = 0) return n

### 修改密码：

进入neo4j提供的可视化界面broswer 输入： `:server change-password`  
键入原密码及新密码，即可修改

### 关于Neo4j和Cypher批量更新和批量插入优化的5个建议

[https://blog.csdn.net/hwz2311245/article/details/60963383](https://blog.csdn.net/hwz2311245/article/details/60963383)

### Neo4j之Cypher学习总结

[https://www.jianshu.com/p/2bb98c81d8ee](https://www.jianshu.com/p/2bb98c81d8ee)
﻿




# neo4j安装指导和Cypher语句





- [neo4j安装指导和Cypher语句](#neo4j安装指导和cypher语句)
    - [下载与安装](#下载与安装)
        - [1.Neo4j ubuntu安装教程](#1neo4j-ubuntu安装教程)
        - [1.创建节点](#1创建节点)
        - [2.查询节点](#2查询节点)
            - [2.1 查询整个图形数据库](#21-查询整个图形数据库)
            - [2.2 查询具有指定标签的节点](#22-查询具有指定标签的节点)
            - [2.3 where 谓词查询](#23-where-谓词查询)
        - [3.创建关系](#3创建关系)
            - [3.1 创建没有任何属性的关系](#31-创建没有任何属性的关系)
            - [3.2 创建关系，并设置关系的属性](#32-创建关系并设置关系的属性)
            - [3.3 创建双向关系](#33-创建双向关系)
        - [4 查询关系](#4-查询关系)
            - [4.1 查询整个数据图形](#41-查询整个数据图形)
            - [4.2 查询跟指定节点有关系的节点](#42-查询跟指定节点有关系的节点)
            - [4.3，查询有向关系的节点](#43查询有向关系的节点)
            - [4.4 为关系命名，通过\[r\]为关系定义一个变量名，通过函数type获取关系的类型](#44-为关系命名通过\r\为关系定义一个变量名通过函数type获取关系的类型)
            - [4.5 查询特定的关系类型，通过\[Variable:RelationshipType{Key:Value}\]指定关系的类型和属性](#45-查询特定的关系类型通过\variablerelationshiptypekeyvalue\指定关系的类型和属性)
        - [5 删除](#5-删除)
            - [5.1删除节点](#51删除节点)
            - [5.2 删除关系](#52-删除关系)
            - [5.3 强制删除节点和关系](#53-强制删除节点和关系)
        - [6 常用查询关键词](#6-常用查询关键词)
            - [6.1 count](#61-count)
            - [6.2 Limit](#62-limit)
            - [6.3 Distinct](#63-distinct)
            - [6.4 Order by](#64-order-by)
            - [6.5 根据id查找](#65-根据id查找)
            - [6.6 In的用法](#66-in的用法)
            - [6.7 Exists](#67-exists)
            - [6.8 With](#68-with)
            - [6.9 Contains](#69-contains)
            - [6.10 Union all (Union)](#610-union-all-union)
        - [7\. 更新](#7\-更新)
            - [7.1 创建一个完整的Path](#71-创建一个完整的path)
            - [7.2 为节点增加一个属性](#72-为节点增加一个属性)
            - [7.3 为节点增加标签](#73-为节点增加标签)
            - [7.4 为关系增加属性](#74-为关系增加属性)
            - [7.5 MERGE](#75-merge)
            - [7.6 跟实体相关的函数](#76-跟实体相关的函数)
        - [8.跟索引相关的函数](#8跟索引相关的函数)
            - [8.1创建索引](#81创建索引)
            - [8.2查询索引](#82查询索引)
            - [8.3删除索引](#83删除索引)
            - [8.4 备份与导入](#84-备份与导入)
        - [1.使用with关键字](#1使用with关键字)
        - [2.直接拼接关系节点查询](#2直接拼接关系节点查询)
        - [4.使用深度运算符](#4使用深度运算符)
        - [使用APOC库](#使用apoc库)
        - [[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)合并重复节点](#httpsneo4j-contribgithubioneo4j-apoc-procedureshttpsneo4j-contribgithubioneo4j-apoc-proceduresexport-importabrimg-src合并重复节点)
        - [安装apoc](#安装apoc)
        - [复制领域](#复制领域)
        - [keys函数](#keys函数)
        - [修改密码：](#修改密码)
        - [关于Neo4j和Cypher批量更新和批量插入优化的5个建议](#关于neo4j和cypher批量更新和批量插入优化的5个建议)
        - [Neo4j之Cypher学习总结](#neo4j之cypher学习总结)

## 下载与安装

### 1.Neo4j ubuntu安装教程

**首先使用**：
Debian repository:

```bash
     wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add - echo 'deb https://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list

    sudo apt-get update
```

**之后选择社区版安装**：

```bash
    sudo apt-get install neo4j
```

安装可能比较慢，别急

**然后配置基本信息**：

```bash
      cd /etc/neo4j
      sudo gedit neo4j.conf
```

进入文档将50行前面的注释去掉
dbms.connector.bolt.listen_address=0.0.0.0:7687

进入文档将第54行前面的注释去掉
dbms.connectors.default_listen_address=0.0.0.0：7474

保存之后，便可以重启
重启命令：neo4j restart
不报错就可以直接使用了

**如果此时报错**，没有这个文件或者文件夹

那就执行以下操作

```bash
mkdir /var/run/neo4j
neo4j start
```

之后在浏览器上访问
<服务器ip地址>:7474/browser/即可看到，初始账户和密码为neo4j，之后自己修改

Active database: graph.db  
Directories in use:  
  home:         /var/lib/neo4j  
  config:       /etc/neo4j  
  logs:         /var/log/neo4j  
  plugins:      /var/lib/neo4j/plugins  
  import:       /var/lib/neo4j/import  
  data:         /var/lib/neo4j/data  
  certificates: /var/lib/neo4j/certificates  
  run:          /var/run/neo4j  
Starting Neo4j.

### 1.创建节点

创建Person 标签,刘德华等若干节点,各自有name,birthday ,born,englishname等属性

1. `match(a),(b) where a.uuid=5110 and b.uuid=12110 create (a)-[r:父亲 {name:"父亲",properties:{relationID:521,time:"1997-05-12"},relationID:521}]->(b)`
2. `create (n:Person { name: '朱丽倩', birthday:'1966年4月6日',born: 1966 ,englishname:'Carol'}) ;`
3. `create (n:Person { name: '刘向蕙', birthday:'2012年5月9日',born: 2012 ,englishname:'Hanna'}) ;`
4. `create (n:Person { name: '任贤齐', birthday:'1966年6月23日',born: 1966 ,englishname:'Richie Jen'}) ;`
5. `create (n:Person { name: '金城武', birthday:'1973年10月11日',born: 1973,englishname:'Takeshi Kaneshiro'}) ;`
6. `create (n:Person { name: '林志玲', birthday:'1974年11月29日',born: 1974,englishname:'zhilin'}) ;`
7. `创建Movie 标签,彩云曲等若干节点,各自有title,released 等属性`
8. `create (n:Movie { title: '彩云曲',released: 1981})`
9. `create (n:Movie { title: '神雕侠侣',released: 1983})` 
10. `create (n:Movie { title: '暗战',released: 2000})`
11. `create (n:Movie { title: '拆弹专家',released: 2017})`

### 2.查询节点

#### 2.1 查询整个图形数据库

点击节点，查看节点的属性，如图，Neo4j自动为节点设置ID值  
`match(n) return n;`

![](http://file.miaoleyan.com/nndt/RTLNZTpcpXFmXkqEFgghgOSCquXrX6Lb)

#### 2.2 查询具有指定标签的节点

查询Movie标签下的节点  
`match(n:Movie) return n;`  
![](http://file.miaoleyan.com/nndt/ONuJJJ9L6T3qUoaJ0OBNf2GzB7lELvsr)

#### 2.3 where 谓词查询

查询名称为林志玲的节点  
`match (n:Person) where n.name='林志玲' return n`  
![](http://file.miaoleyan.com/nndt/X7DxApWede4g8tTxWKMYcQQmoOsLQOLt)
(查询具有指定属性的节点)结果相同  
`match(n{name:'林志玲'}) return n;`  
![](http://file.miaoleyan.com/nndt/OPQJDR5m291S7ERXmzSAdFVFRIYclSuL)
查询born属性小于1967的节点  
`match(n) where n.born<1967 return n;`

### 3.创建关系

关系的构成：StartNode - \[Variable:RelationshipType{Key1:Value1,Key2:Value2}\] -> EndNode，在创建关系时，必须指定关系类型。

#### 3.1 创建没有任何属性的关系

1. `MATCH (a:Person),(b:Movie)`
2. `WHERE a.name = '刘德华' AND b.title = '暗战'`
3. `CREATE (a)-[r:DIRECTED]->(b)`
4. `RETURN r;`

![](http://file.miaoleyan.com/nndt/Fqjr5iaDpgDOQg7LduUKIhFzaysqPHtZ)

#### 3.2 创建关系，并设置关系的属性

1. `MATCH (a:Person),(b:Movie)`
2. `WHERE a.name = '刘德华' AND b.title = '神雕侠侣'`
3. `CREATE (a)-[r:出演 { roles:['杨过'] }]->(b)`
4. `RETURN r;`

![](http://file.miaoleyan.com/nndt/NV8VSDingSlUjsLnjL3THI37R3CuukLx)

#### 3.3 创建双向关系

刘德华的女是刘向蕙,刘向蕙的父亲是刘德华

1. `MATCH (a:Person),(c:Person)`
2. `WHERE a.name = '刘德华' AND c.name = '刘向蕙'`
3. `CREATE (a)-[r:父亲 { nickname:'甜心' }]->(c),`
4. `(c)-[d:女儿 { nickname:'爹地' }]->(a)`
5. `RETURN r;`

![](http://file.miaoleyan.com/nndt/KwSusLqS5USIc9ZtT81Xo1A8hs9IqEiS)  
关系建错了 删除关系 (见5.2 )  
重新创建

1.  `MATCH (a:Person),(c:Person)`
2.  `WHERE a.name = '刘德华' AND c.name = '刘向蕙'`
3.  `CREATE (a)-[d:女儿 { nickname:'甜心' }]->(c),`
4.  `(c)-[r:父亲 { nickname:'爹地' }]->(a)`
5.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/dPKmVIkfEp88kf0IPqvlkhxzn0Ifu2aX)

1.  `MATCH (a:Person),(c:Movie)`
2.  `WHERE a.name = '刘德华' AND c.title = '彩云曲'`
3.  `CREATE (a)-[r:出演 { partner:'张国荣' }]->(c),`
4.  `(c)-[d:演员 { rolename:'阿华哥' }]->(a)`
5.  `RETURN r;`
6.  `MATCH (a:Person),(c:Movie)`
7.  `WHERE a.name = '刘德华' AND c.title = '拆弹专家'`
8.  `CREATE (a)-[r:出演 { partner:'赵薇,高圆圆' }]->(c),`
9.  `(c)-[d:演员 { rolename:'华仔' }]->(a)`
10.  `RETURN r;`
11.  `MATCH (a:Person),(c:Movie)`
12.  `WHERE a.name = '刘德华' AND c.title = '神雕侠侣'`
13.  `CREATE (a)-[r:出演 { partner:'刘亦菲' }]->(c),`
14.  `(c)-[d:演员 { rolename:'杨过' }]->(a)`
15.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/tRgmHlikkn2GYfIiZF2FrKgpEbxlnrWN)

1.  `继续新增关系 刘德华和林志玲,金城武,任贤齐`
2.  `MATCH (a:Person),(c:Person)`
3.  `WHERE a.name = '刘德华' AND c.name = '任贤齐'`
4.  `CREATE (a)-[d:朋友 { sex:'男' }]->(c)`
5.  `RETURN d;`

7.  `MATCH (a:Person),(c:Person)`
8.  `WHERE a.name = '刘德华' AND c.name = '金城武'`
9.  `CREATE (a)-[d:朋友 { sex:'男' }]->(c)`
10.  `RETURN d;`
11.   `这里没有给关系设置属性sex`
12.  `MATCH (a:Person),(c:Person)`
13.  `WHERE a.name = '刘德华' AND c.name = '林志玲'`
14.  `CREATE (a)-[d:朋友]->(c)`
15.  `RETURN d;` 

17.  `查询Person表关系`
18.  `MATCH (n:Person) RETURN n`

![](http://file.miaoleyan.com/nndt/pBQJwvrnzIAN1eUV6oNFZMutUjl0eHhH)

### 4 查询关系

在Cypher中，关系分为三种：符号“—”，表示有关系，忽略关系的类型和方向；符号“—>”和“<—”，表示有方向的关系；

#### 4.1 查询整个数据图形

`match(n) return n;`

![](http://file.miaoleyan.com/nndt/8Gy521m3erXIR320B7X2KYp73VQ1cKvn)

#### 4.2 查询跟指定节点有关系的节点

查询跟Movie标签有关系的所有节点

![](http://file.miaoleyan.com/nndt/W9gtIB9wA7BxJGGWhsfT4pQvZpdpmjbK)

#### 4.3，查询有向关系的节点

查询和刘德华有关系的电影  
`MATCH (:Person { name: '刘德华' })-->(movie)RETURN movie;`

![](http://file.miaoleyan.com/nndt/xbfKJHuNPq9oZ46rwNSbYKQtAuyLV874)

#### 4.4 为关系命名，通过\[r\]为关系定义一个变量名，通过函数type获取关系的类型

1.  `MATCH (:Person { name: '刘德华' })-[r]->(movie) RETURN r,type(r);`

![](http://file.miaoleyan.com/nndt/nrhktZohsHEQhoMdOcAwV6Fh10As7Z30)

#### 4.5 查询特定的关系类型，通过\[Variable:RelationshipType{Key:Value}\]指定关系的类型和属性

`MATCH (:Person { name: '刘德华' })-[r:出演{partner:'张国荣'}]->(Movie) RETURN r,type(r);`

![](http://file.miaoleyan.com/nndt/Nlc3Fii8HQGq4LTaM3870TyCUICxnGcu)  
查询和刘德华和张国荣合作过的电影  
`MATCH (:Person { name: '刘德华' })-[r:出演{partner:'张国荣'}]->(m:Movie) RETURN m;`

![](http://file.miaoleyan.com/nndt/wfEwwVuu2FenkA5EePSBls6byYHW3c9m)  
查询被刘德华称呼为甜心的女儿  
`MATCH (:Person { name: '刘德华' })-[r:女儿{nickname:'甜心'}]->(m:Person) return m`  
查询刘德华的老婆是谁  
`Match (n:Person{name: '刘德华'})-[:wife]->(a:Person) return a`

![](http://file.miaoleyan.com/nndt/2Cjh7lcfJGmM1RZCwcj1BlFSpyVaDmlX)  
刘德华出演过的电影

![](http://file.miaoleyan.com/nndt/jXg8WZcEBwqggczfGeO9eqzfAqcBgrFw)

### 5 删除

#### 5.1删除节点

1.  `create (n:City { name: '北京'})`
2.  `Match (n:City{name:'北京'}) delete n`

#### 5.2 删除关系

1.  `Match (a:Person{name:'刘德华'})-[r:父亲]->(b:Person{name:'刘向蕙'}) delete r`
2.  `Match (a:Person{name:'刘向蕙'})-[r:女儿]->(b:Person{name:'刘德华'}) delete r`

#### 5.3 强制删除节点和关系

1.  ``Match (n:`美国军事基地`) where n.name ='挂载' detach delete n``

### 6 常用查询关键词

#### 6.1 count

查询Person 一共有多少人  
`Match (n:Person ) return count(n)`

![](http://file.miaoleyan.com/nndt/zIhhZ6gGOJn6Hc53fY9Ighs2OMS35yvR)  
查询标签(Person)中born=1966的一共有多少节点（人）：  
三种写法(第三种不能用似乎)：  
`1. Match (n:Person) where n.born=1966 return count(n)`

![](http://file.miaoleyan.com/nndt/AML1BD0lwnn1JDatuIAUwrW2kxDtRT9i)

1.  Match (n:Person{born:1966}) return count(n)//特别注意类型,如果存的类似是数字类型,使用字符串就查不出来,如下:

![](http://file.miaoleyan.com/nndt/4iF1zmuD74Q4YOHN1NbmOYmZ1p4gEtex)

![](http://file.miaoleyan.com/nndt/sGlRc1ZF1Op3JOe9ZfArHfh5iSuxHNNY)

#### 6.2 Limit

`Match (n:Person) return n limit 3`

![](http://file.miaoleyan.com/nndt/rHVr6899KoS5TKkXprShyib7AYo4YFYS)

![](http://file.miaoleyan.com/nndt/nbc9h3b1wsWzE4dvAGqokWNlOvqqwmSJ)

#### 6.3 Distinct

`Match (n:Person) return distinct(n.born)`

![](http://file.miaoleyan.com/nndt/Q3SWdrVwcPEX9FoFdtFU5Cti9GwZ6tn2)

#### 6.4 Order by

1.  `Match(n:Person) return n  order by n.born (默认升序)`
2.  `Match(n:Person) return n  order by n.born  asc (升序)`
3.  `Match(n:Person) return n  order by n.born  desc (降序)`

![](http://file.miaoleyan.com/nndt/j7yfWb7E5WiQQMqanHN0QhROengjMy3o)

![](http://file.miaoleyan.com/nndt/Ulj7j0vsdA3li6jFDQSQYF4bB7wINi0L)

![](http://file.miaoleyan.com/nndt/zBZgSDBc6lGh0Qhp1RiUYcPUHJSMrWBV)

#### 6.5 根据id查找

`match (n) where id(n)=548 return n`

![](http://file.miaoleyan.com/nndt/jN65PxJkcKdLZMHBCnH9d8dIVoxyqqNb)

#### 6.6 In的用法

1.  `Match (n) where ID(n) IN[353,145,547] return n` 
2.  `Match (n) where ID(n) IN[145,175,353,547,548] return n`

![](http://file.miaoleyan.com/nndt/tburFLmcTk8PVUbKgGesQc4tCktk4LIe)

#### 6.7 Exists

节点存在 name这个属性的记录：  
`Match (n) where exists(n.title) return n`

![](http://file.miaoleyan.com/nndt/uOyPBc2F2iQBG3rloKiCRlBPqpXAIvSo)

#### 6.8 With

查询name以‘刘’开头的节点：  
`Match (n) where n.name starts with '刘' return n`

![](http://file.miaoleyan.com/nndt/pbadSnGVkvqa5ZVHgLT3ALVcE5MICy6m)

#### 6.9 Contains

查询title中含有 ‘侠侣’的节点  
`Match (n:Movie) where n.title Contains '侠侣' return n`

![](http://file.miaoleyan.com/nndt/06SgibFRfiOkCIWotMNJ7y8GSRmsBCKZ)

#### 6.10 Union all (Union)

求并集，不去重（去重用Union, as 取别名）:

1.  `Match(n:Person) where n.born=1966 return n.name as name`
2.  `Union all`
3.  `Match(n:Movie) where n.released=1983 return n.title as name`

![](http://file.miaoleyan.com/nndt/Hl1koSGFVR9S3cCQfeAIz016YDXo7JQc)

### 7\. 更新

#### 7.1 创建一个完整的Path

1.  `CREATE p =(m:Person{ name:'刘亦菲',title:"演员" })-[:签约]->(neo)<-[:签约]-(n:Person { name: '赵薇',title:"投资人" })`
2.  `RETURN p`

![](http://file.miaoleyan.com/nndt/3ZsdZBQ0oLWracGYir7T1r66zUZ3unWX)

![](http://file.miaoleyan.com/nndt/r4vd7YLQb6C8pP5JS7XEHymLvAQ40QOE)

#### 7.2 为节点增加一个属性

通过节点的ID获取节点，Neo4j推荐通过where子句和ID函数来实现。

1.  `match (n)`
2.  `where id(n)=358`
3.  `set n.name = '华谊兄弟'`
4.  `return n;`

为节点移除属性  
`MATCH (n:农业) where id(n)=17816137 remove n.sortcode,n.targettable,n.unit,n.uuid`

![](http://file.miaoleyan.com/nndt/n9c6OehJBPF6outqWhx2ZBY73XKIt0ZC)

#### 7.3 为节点增加标签

1.  `match (n)`
2.  `where id(n)=358`
3.  `set n:公司`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/JKUqi66mJCfU500VOEKYEG3YDOTpZ1bS)

#### 7.4 为关系增加属性

1.  `match (n)-[r]->(m)`
2.  `where id(n)=357 and id(m)=358`
3.  `set r.经纪人='程晨'`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/wKCcFjUsfqNwD54OwqD4nPHXuROlw11F)  
此时图谱效果成

![](http://file.miaoleyan.com/nndt/I3CPxRfHSDEGvoJubB9Abf5WYiN9hhhI)  
接着让刘德华也和华谊兄弟签约

1.  `MATCH (a:Person),(c:公司)`
2.  `WHERE a.name = '刘德华' AND c.name = '华谊兄弟'`
3.  `CREATE (a)-[d:签约 { 经纪人:'刘得得' }]->(c)`
4.  `RETURN d;`

![](http://file.miaoleyan.com/nndt/FvG0DsSsHtRyFCgHVT3118ILcNeFvMPW)

#### 7.5 MERGE

Merge子句的作用有两个：当模式（Pattern）存在时，匹配该模式；当模式不存在时，创建新的模式，功能是match子句和create的组合。在merge子句之后，可以显式指定on creae和on match子句，用于修改绑定的节点或关系的属性。  
通过merge子句，你可以指定图形中必须存在一个节点，该节点必须具有特定的标签，属性等，如果不存在，那么merge子句将创建相应的节点。  
通过merge子句匹配搜索模式  
匹配模式是：一个节点有Person标签，并且具有name属性；如果数据库不存在该模式，那么创建新的节点；如果存在该模式，那么绑定该节点；

1.  `MERGE (m:Person { name: '迈克尔·杰克逊' }) RETURN m;`

![](http://file.miaoleyan.com/nndt/CmWHpmopMlMCRvMpv2FFcD1GrXWeeuAM)  
在merge子句中指定on create子句  
如果需要创建节点，那么执行on create子句，修改节点的属性；

1.  `MERGE (m:Person { name: '杰森·斯坦森' })`
2.  `ON CREATE SET m.registertime = timestamp()`
3.  `RETURN m.name, m.registertime`

![](http://file.miaoleyan.com/nndt/bi4rNu0KGVPuwgkP6EpOTYjUT8srcuMD)  
在merge子句中指定on match子句  
如果节点已经存在于数据库中，那么执行on match子句，修改节点的属性；节点属性不存在则新增

1.  `MERGE (m:Person)`
2.  `ON MATCH SET m.registertime = timestamp()`
3.  `RETURN m.name, m.registertime`

![](http://file.miaoleyan.com/nndt/wNflMULIU6uB4xLBrfJpiVMzwirf0Lf5)  
在merge子句中同时指定on create 和 on match子句(没有对应属性则修改不成功,不会新增属性)

1.  `MERGE (m:Person{ name: '李连杰' })`
2.  `ON CREATE  SET m.registertime = timestamp()`
3.  `ON MATCH SET m.offtime = timestamp()`
4.  `RETURN m.name, m.registertime,m.offtime`

![](http://file.miaoleyan.com/nndt/3XD3diqzwUsFiu1zszZNkkKWWEQ0eWmV)  
merge子句用于match或create一个关系

1.  `MATCH (m:Person { name: '刘德华' }),(n:Movie { title: '神雕侠侣' })`
2.  `MERGE (m)-[r:导演]->(n)`
3.  `RETURN m.name, type(r), n.title`

![](http://file.miaoleyan.com/nndt/GJFoWbQaleDaQGS45AK2U6UWfHVAUOQg)  
merge子句用于match或create多个关系  
赵薇既是神雕侠侣的导演,也是神雕侠侣的演员

1.  `MATCH (m:Person { name: '赵薇' }),(n:Movie { title: '神雕侠侣' })`
2.  `MERGE (m)-[r:导演]->(n)<-[r2:出演]-(m)`
3.  `RETURN m.name, type(r),type(r2), n.title`

![](http://file.miaoleyan.com/nndt/b2cxKGhpGRhdstnPb3W4I5lpezii9Zok)  
merge子句用于子查询  
先添加基础数据  
创建城市节点

1.  `create (n:City { name: '北京',othername:'帝都'})`
2.  `create (n:City { name: '香港',othername:'HongKong'})`
3.  `create (n:City { name: '台湾',othername:'湾湾'})`

为Person标签的每个节点都增加一个属性 bornin

1.  `match (n:Person)`
2.  `set n.bornin = ''`
3.  `return n;`

![](http://file.miaoleyan.com/nndt/Fbkg8VXyxHwQj4pWG9VGVjv7cw4sXx46)  
match (n)  
where id(n)=175  
set n.bornin = ‘香港’  
return n;

![](http://file.miaoleyan.com/nndt/US8xuWIAsjlshqekJiLaAt4bnYKCkfNh)

1.  `match (n)`
2.  `where n.name='金城武'`
3.  `set n.bornin = '台湾'`
4.  `return n;`

![](http://file.miaoleyan.com/nndt/32KRi3ietIMC8mHwOz7B6zRy1wM2qZbF)  
需求:查找刘德华和金城武的信息和所在地的othername(相当于mysql 连表查询)

1.  `MATCH (p:Person)`
2.  `where p.name='刘德华' or p.name='金城武'`
3.  `MERGE (c:City { name: p.bornin })`
4.  `RETURN p.name,p.born,p.bornin , c.othername;`

![](http://file.miaoleyan.com/nndt/VzAbRCB535csGqMSWbYbZyRkYtxs5HVd)  
创建刘德华出生地是香港这条关系

1.  `MATCH (a:Person),(c:City)`
2.  `WHERE a.name = '刘德华' AND c.name = '香港'`
3.  `CREATE (a)-[r:出生地]->(c)`
4.  `RETURN r;`

![](http://file.miaoleyan.com/nndt/Epmlnzzllew51X7XaIJs4tyyRlZcDanR)  
需求:给Person中每个节点都创建一个出生地的关系,没有则返回null

1.  `MATCH (p:Person)`
2.  `MERGE (c:City { name: p.bornin })`
3.  `MERGE (p)-[r:出生地]->(c)`
4.  `RETURN p.name, p.bornin, c.othername;`

![](http://file.miaoleyan.com/nndt/7boDFnBp6goe9ZKuEHQ9GvCsbdZN1dQi)

![](http://file.miaoleyan.com/nndt/AN7zmSwWDXkFps1rbFjqERwrzaF6zwfZ)  
删除这些关系

1.  `Match (a:Person)-[r:出生地]->(c:City{name:''}) delete r`
2.  `Match (a:City)-[r:出生地]->(c:Person) delete r`

![](http://file.miaoleyan.com/nndt/yh2ule0aJWUqct24hYhJBkiwUqQAglkm)  
查询Person标签中不存在name属性的节点

1.  `Match (n:Person) where not exists(n.name) return n`
2.  `Match (n:Person) where not exists(n.name) delete n`

![](http://file.miaoleyan.com/nndt/WnvCH8SZSXuT7DJJtxeasCWH7XDVDagB)  
create (n:Prize { name: ‘金马奖’});  
create (n:Prize { name: ‘奥斯卡金奖’});  
create (n:Prize { name: ‘金鸡奖’});  
create (n:Prize { name: ‘香港电影金像奖’});

#### 7.6 跟实体相关的函数

通过id函数，返回节点或关系的ID  
查询Person标签中和刘德华有关系的 id(节点和关系)

`MATCH (:Person { name: '刘德华' })-[r]->(movie) RETURN id(r);`

![](http://file.miaoleyan.com/nndt/H5LdHRTguYNVlMwMQsNVegVNn9gnyNZN)

通过type函数，查询关系的类型  
查询Person标签中和刘德华相关的关系(以下三种结果相同)

1.  `MATCH (:Person { name: '刘德华' })-[r]->(a)`
2.  `MATCH (:Person { name: '刘德华' })-[r]->(b)`
3.  `MATCH (:Person { name: '刘德华' })-[r]->()`
4.  `RETURN type(r);`

![](http://file.miaoleyan.com/nndt/SCfmXdAKG5E4R3aYUozB77HDoZnJF7BS)

![](http://file.miaoleyan.com/nndt/M6pWHBJpoVCjIWBOCrkHeBmwo0iOX8k6)  
通过lables函数，查询节点的标签  
查询和刘德华有关系的节点

1.  `MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN p;`

![](http://file.miaoleyan.com/nndt/HRw9QgLKVedhryFsJARXaERZh1cdrrlx)  
查询和刘德华有关系的标签(去重)

1.  `MATCH (:Person { name: '刘德华' })-[r]->(p)`
2.  `RETURN distinct(labels(p))`

![](http://file.miaoleyan.com/nndt/PCxlOZAJY6MEvfhjxLLcBEBucWUvusDS)  
通过keys函数，查看节点或关系的属性键

1.  `MATCH (a)`
2.  `WHERE a.name = '刘德华'`
3.  `RETURN keys(a)`

![](http://file.miaoleyan.com/nndt/wGYxL2fe4tIGMRSyU5JVuZXcN8KU84yc)  
`MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN distinct(keys(r))`

![](http://file.miaoleyan.com/nndt/NTFr6cZ2lFwpmZKk8tWtQaGdjSlolo3J)  
通过properties()函数，查看节点或关系的属性

1.  `MATCH (a)`
2.  `WHERE a.name = '刘德华'`
3.  `RETURN properties(a)`

![](http://file.miaoleyan.com/nndt/BL6DS5eJSpB13XGATi0nJ4UZmyIl5wyn)  
`MATCH (:Person { name: '刘德华' })-[r]->(p) RETURN properties(r)`

![](http://file.miaoleyan.com/nndt/4yPRBh1PGUqJpHVvC2QHPTcdYE0w6emp)

### 8.跟索引相关的函数

#### 8.1创建索引

创建单一属性索引

1.  `CREATE INDEX ON :Lable(property)`
2.  `Query:`
3.  `CREATE INDEX ON :Person(name)`
4.  `给数据库的:Person标签的name属性创建索引`
5.  `Result:`
6.  `No data returned, and nothing was changed`
7.  `创建复合属性索引`
8.  `CREATE INDEX ON :Label(prop1,...,propN)`
9.  `Query:`
10.  `CREATE INDEX ON :Person(age,country)`
11.  `Result:`
12.  `No data returned.`
13.  `Indexes added:1`

#### 8.2查询索引

CALL db.Indexes

![](http://file.miaoleyan.com/nndt/rCE5mke8jIqPrM5aO8WnHXNCbGVeXUQT)

#### 8.3删除索引

1.  `DROP INDEX ON :LABEL(property）`
2.  `Query:`
3.  `DROP INDEX ON :Person(firstname)`
4.  `Result:`
5.  `No data returned`
6.  `Indexes removed:1`

8.  `DROP INDEX ON :LABEL(prop1,...,propN)`
9.  `Query:`
10.  `DROP INDEX ON :Person(age,country)`
11.  `Result:`
12.  `No data returned`
13.  `Indexes removed:1`

#### 8.4 备份与导入

备份/导出  
要以管理员身份运行  
`neo4j-admin dump --database=graph.db --to=E:\neo4jdata`

![](http://file.miaoleyan.com/nndt/BGvBvFpGCMm5Nn6TlC4dKTGcWmE6vl5L)

![](http://file.miaoleyan.com/nndt/kct9pwEo1HYeZYnWwfwcS0EA3a4yxJFH)  
导入  
`neo4j-admin load --from=E:\neo4jdata\graph.db.dump --database=graph.db --force`

导入csv

1.  ``USING PERIODIC COMMIT 500 LOAD CSV FROM "file:///D:\\test.csv" AS line  MERGE (:`顶顶顶` {name:line[0]})``
2.  `LOAD CSV WITH HEADERS FROM "file:///C:\\Program Files\\Java\\neo4j-community-3.4.7\\import\\stock_concept.csv"`
3.  `AS line`
4.  `return line.n`

![](http://file.miaoleyan.com/nndt/WuOaUz2UvKWkCKAa9c5bz148YZQ7Q53d)  
USING PERIODIC COMMIT 10  
LOAD CSV FROM “file:///node.csv” AS line  
create (a:Node{name:line\[0\]})

![](http://file.miaoleyan.com/nndt/waTKUTyuMYliPrutDWbGhR0hsyTRcOJ3)  
USING PERIODIC COMMIT  
LOAD CSV FROM ‘file:///concept.csv’ AS row  
CREATE (n:概念{name:row\[1\],uuid:row\[0\]})

csv 不带header方式  
USING PERIODIC COMMIT  
LOAD CSV FROM “file:///executive\_stock.csv” AS row  
MATCH (n:高管 {uuid: row\[0\]})  
MATCH (m:企业 {uuid: row\[1\]})  
MERGE (n)-\[:RE{name:row\[2\]}\]->(m)

带header方式  
USING PERIODIC COMMIT  
LOAD CSV WITH HEADERS FROM ‘file:///industry.csv’ AS row  
CREATE (:行业{name:row.name,uuid:row.uuid})

关于neo4j查询多深度关系节点
----------------

### 1.使用with关键字

1.   `查询三层级关系节点如下：with可以将前面查询结果作为后面查询条件`
2.  `match (na:company)-[re]->(nb:company) where na.id = '12399145' WITH na,re,nb match (nb:company)-[re2]->(nc:company) return na,re,nb,re2,nc`

### 2.直接拼接关系节点查询

`match (na:company{id:'12399145'})-[re]->(nb:company)-[re2]->(nc:company) return na,re,nb,re2,nc`

3.为了方便，可以将查询结果赋给变量，然后返回
-----------------------

`match data=(na:company{id:'12399145'})-[re]->(nb:company)-[re2]->(nc:company) return data`

### 4.使用深度运算符

当实现多深度关系节点查询时，显然使用以上方式比较繁琐。  
可变数量的关系->节点可以使用-\[:TYPE_minHops..maxHops\]->。  
查询：  
如果在1到3的关系中存在路径，将返回开始点和结束点。  
\`match data=(na:company{id:’12399145’})-\[_1..3\]->(nb:company) return data\`

### 使用APOC库

[https://neo4j-contrib.github.io/neo4j-apoc-procedures/](https://neo4j-contrib.github.io/neo4j-apoc-procedures/)  
导入导出的几种方式  
[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)

[](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)

### [](https://neo4j-contrib.github.io/neo4j-apoc-procedures/>https://neo4j-contrib.github.io/neo4j-apoc-procedures/#export-import</a><br><img src=)合并重复节点

### 安装apoc

过程如下  
[https://blog.csdn.net/graphway/article/details/78957415](https://blog.csdn.net/graphway/article/details/78957415)  
安装完成 执行 return apoc.version() 查看下版本  
![](http://file.miaoleyan.com/nndt/ipPR41bCgMDDeT92Jcsu2KGQU4dFq0BB)  
MATCH (n:国家电网)  
WITH n.name AS name, COLLECT(n) AS nodelist, COUNT(\*) AS count  
WHERE count > 1  
CALL apoc.refactor.mergeNodes(nodelist) YIELD node  
RETURN node

查询某个节点有关系的3级及以内的路径  
MATCH (n:`贵州`) WHERE n.name=’交通事件’  
CALL apoc.path.spanningTree(n, {maxLevel:3}) YIELD path  
RETURN path;

### 复制领域

1.  `match(n:菊花) MERGE (:大萨达{name:n.name})`
2.  `match(n:菊花)-[r]->(q:菊花)`
3.  `with n, r, q`
4.  `match (o:大萨达{name:n.name}), (m:大萨达{name:q.name})`
5.  `MERGE (o)-[:RE{name:r.name}]->(m)`

### keys函数

1.  `match(n) where any(x in keys(n) where n[x] > 0) return n 查询某个属性大于0 的节点`
2.  `match(n) where all(x.uuid in keys(n) where n[x.uuid] > 0) return n 查询所有属性大于0的节点`

例如:x在any中是一个变量，并不是属性  
match(n) where any(uuid in keys(n) where n\[uuid\] > 0) return n uuid大于0  
match(n) where all(uuid in keys(n) where n\[uuid\] > 0) return n 所有uuid都大于0  
match(n) where any(querytype in keys(n) where n\[querytype\] = 0) return n

### 修改密码：

进入neo4j提供的可视化界面broswer 输入： `:server change-password`  
键入原密码及新密码，即可修改

### 关于Neo4j和Cypher批量更新和批量插入优化的5个建议

[https://blog.csdn.net/hwz2311245/article/details/60963383](https://blog.csdn.net/hwz2311245/article/details/60963383)

### Neo4j之Cypher学习总结

[https://www.jianshu.com/p/2bb98c81d8ee](https://www.jianshu.com/p/2bb98c81d8ee)
