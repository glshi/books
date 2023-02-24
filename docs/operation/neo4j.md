## neo4j导入数据

**五种导入方式**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303222959241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29ucm9hZGxpdXlhcWlvbmc=,size_16,color_FFFFFF,t_70)

**方式一：load csv 方式**



进入$NEO_HOME/conf/neo4j.conf配置文件并取消这一行的注释：dbms.directories.import=import  

开启引入文件

```
apoc.import.file.enabled=true
dbms.security.procedures.unrestricted=apoc.*
dbms.security.allow_csv_import_from_file_urls=true
```

导入的文件必须是csv文件，位置可以是本地的，或通过http、https、ftp等url指定位置。

配置文件路径在  neo4j/conf/neofj.conf。

neo4j导入文件的设置是dbms.security.allow_csv_import_from_file_urls，默认为true；

而导入本地文件的位置通过dbms.directories.import来指定导入的根目录，默认目录是neo4j/import目录下,然后再使用file:///来表示绝对路径。



**因为neo4j是utf-8的，而CSV默认保存是ANSI的，需要用记事本另存为成UTF-8。**



1）、不带header,用下标来索引  
如给定artists.csv文件

```
LOAD CSV FROM 'https://neo4j.com/docs/cypher-manual/artists.csv' AS line
CREATE (:Artist { name: line[1], age:line[2]})
```

2）、带header，用关键字来索引  
如给定给定artists-with-headers.csv文件

```
LOAD CSV WITH HEADERS FROM 'https://neo4j.com/docs/cypher-manual/artists-with-headers.csv' AS line
CREATE (:Artist { name: line.Name, age:line.Age})
```

3)、大csv文件分批导入

```
USING PERIODIC COMMIT
LOAD CSV FROM 'https://neo4j.com/docs/cypher-manual/artists.csv' AS line
CREATE (:Artist { name: line[1], age:line[2]})
```

这里默认1000行提交一次，也可以人为指定，比如using periodic commit 500。如果值中包含引号，可以用"“来表示”。使用load csv只能导入结点，如果还想导入关系数据，就只能靠neo4j自带的import工具了。



**从文件导入示例：**

```shell
# 导入实体
LOAD CSV WITH HEADERS  FROM "file:///zcy.csv" AS line
MERGE (z:中成药{name:line.name})
```

```shell
# 导入实体
LOAD CSV WITH HEADERS  FROM "file:///herber.csv" AS line
MERGE (z:中草药{name:line.name})
```

```shell
# 导入关系第一种方法：
LOAD CSV WITH HEADERS FROM "file:///r_contain.csv" AS row  
match (from:中成药{name:row.from}),(to:中草药{name:row.to})  
merge (from)-[r:主要成分{property:row.property}]->(to)
```

```shell
# 导入关系第二种方法：
LOAD CSV WITH HEADERS FROM "file:///r_contain.csv" AS line  
match (from:中成药{name:line.from}),(to:中草药{name:line.to})  
merge (from)-[r:主要成分{property:line.property}]->(to)
```

**方式二：neo4j-admin import**

通过neo4j-admin方式导入的话，需要暂停服务，并且需要清除graph.db,这样才能导入进去数据。而且，只能在初始化数据时，导入一次之后，就不能再次导入。所以这种方式，可以在初次建库的时候，导入大批量数据，等以后如果还需要导入数据时，可以采用上边的方法。
