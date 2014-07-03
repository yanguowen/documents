# SAP HANA
HANA:
grant select on schema <SCHEMA> to _SYS_REPO with grant option;  
with parameters ('request_flags'='analyze_model');  

float(n), n < 25为real,否则为double  
varchar/nvarchar最大为5000  
alphanum为字符和字母  
shorttext/text不是独立类型，生成一个nvarchar或者NCLOB，可用来查找等(系统缺省建立了查找索引)  
LOB最大为2G  
varchar中英文为一个字节，中文为2个，nvarchar不论中英文都为2个字节  
N'中文'unicode常量前面加个N  
十六进制数字为0x前缀，如0x0abc  
二进制字符串为X前缀，如X'0abcd'  

用户也可以在char/varchar等列上显示建立查找索引  
shorttext/text不是独立类型，生成一个nvarchar或者NCLOB，可用来查找等(系统缺省建立了查找索引)  

Text Search:  
在很多字段上,不止是text字段(非text字段需要手动建立全文索引),关键字contains  
Select highlighted(text1) from texttable where contains(text1,’SAP’) 会在SAP上加<b></b>  
Select language(text1) from texttable where contains(text1,’SAP’) 会返回en  

Fuzzy Search:  
模糊搜索 select * from T where contains(column1, 'catz', FUZZY(0.8))  
自由搜索 select * from T where CONTAINS( (column1,column2,column3), 'cats OR dogz', FUZZY(0.7))  


HANA GIS:  
`
CREATE COLUMN TABLE locations
(
id INTEGER,
description CHAR(100),
location ST_POINT
);`
`SELECT description FROM location WHERE location.ST_Within(ST_FromText(‘ST_POLYGON(-1 -1, 2 -1, 2 2, -1 2)’))`  


`cast(6 as varchar)`  
`TO_bigint/...`  
TO_DATS (d)转换为abap中的时间格式  

`SELECT EXTRACT (YEAR FROM TO_DATE ('2010-01-04', 'YYYY-MM-DD')) "extract" FROM DUMMY`  

CEIL(n)返回大于或者等于n 的第一个整数。  

FLOOR (n)返回不大于参数n 的最大数。  


GLOBAL TEMPORARY  
表定义全局可见，但数据只在当前会话中可见。表在会话结束后截断。  

LOCAL TEMPORARY  
表的定义和数据只在当前会话可见，该表在会话结束时被截断。  

`CALL with overview`:结果写在物理表里并返回  


变量引用为:var  
标量赋值为:=  

`IF ELSEIF ENDIF
WHILE DO END WHILE
FOR IN DO END FOR
LOOP END LOOP
BREAK/COUNTINUE`  

EXEC执行动态 SQL  

创建表C1，与表A 的定义相同，记录也相同。  
`CREATE COLUMN TABLE C1 LIKE A WITH DATA;`  

HANA支持更新视图  

RUNCATE 比DELETE FROM 快，但是TRUNCATE无法回滚  

FOR UPDATE 关键字锁定记录，以便其他用户无法锁定或修改记录，直到本次事务结束。  

HANA有2个地方放表，一个是delta，一个是main，delta为写优化，main为读优化，所以在读取的时候需要将delta合并到main里，以提升性能  

HANA的锁： 
数据库锁，表锁，行锁，其中行锁没有读锁，因为用了mvcc模式，但是数据库和表所都有读写锁  

HANA并发：MVCC  
MVCC可以保证读的一致性，但不能保证写冲突，所以需要写锁来保证  

select for update显示要求写锁（行级别）  
写冲突（一个先启动的事务改动了一个后启动先结束事务改过的值）会被检测并爆出错误  

行表可以被一直加载，也可以选择需要的时候被换出，也可以在启动的时候不加载  
行表即使很小也会占用64M内存  

DELTA MERGE是一个表一个表来得  
Smart merge(globle.ini)可以决定在有些时候不merge,如load大表的时候。  
在结束后会把整个内存写到硬盘上(Backup)  
用memory-only可以防止delta merge备份数据  

merge delta of table1:指示强制merge， 但系统会根据自愿情况，不一定立即merge  
merge delat of table1 with parameters ('FORCED_MERGE' = 'ON') 会立即merge  
...whit parameter ('MEMORY_MERGE' = 'ON') 只merge而不写到硬盘上，对于导入后merge比较合适   
Critical merge可以在merge关掉的时候在系统认为需要的时候自动merge  

内存不够用的时候会自动把列数据unload到硬盘上，根据最少使用原则  
Hybrid LOB:可以把LOB数据放在硬盘上,不放到内存的列  
LOB(大对象数据)只加载应用，数据只有真正访问的时候才加载到内存  


隔离机制:
READ COMMITTED | REPEATABLE READ | SERIALIZABLE(后两种为事务快照隔离)  
HANA缺省为重复读  

temporal tables是历史表，可以做time travelling, 记录了所有的更改，有2个隐藏列，validfrom/validto, 数据保存在特定的区域
普通表会在一段时间删除过时的数据  

HANA的update操作是插入一条新记录  

HANA中可以hybird存储，及按列，然后某几列存在一起（像行一样），以增加性能  

HANA load/insert数据时1.5M records/core/s  
HANA读取数据一般3M/ms/core   
CPU/RAM吞吐量是25G/S  

Index就是按照某列进行排序的好的数据，所以可以很快找到那里，不用全表扫描,HANA里也有索引  
缺省列表没有索引而是全列扫描，但是可以建立反向索引以提高性能(From HANA Bluebook)  

`Create column view ... type join` 可以创建join view  

Semi-join  
Hash-join  
Sort-merge join  

不等于join会导致将列表转为行表  

Native Mixed Join可以join行表和列表而不用转换  

UNION和AND可以促使HANA的并行查询  

DELTA里面属性矢量没有压缩（为了快速插入）,B+树建立索引（更快查询）  

2个队列，一个用于OLAP, 一个OLTP  

插入式，先插入到一个行表中，然后行表合并到delta中(列表)，最后delta合并到main  

LOG分2部分，数据表和字典  

列里存的valudId指向字典条目，valudId用最少的位表示，log2(N)位  
主存里的字典是排序的，delta里面没有，为了方便插入  
ValudId序列会被压缩，如prefix endcoding, run length encoding, cluster encoding, sparse encoding, indirect encoding等  
HANA会根据数据分布来选择压缩方式，但是也可以在表里自己指定  
主存字典里的值也会被压缩  

MERGE的时候把数据写到磁盘上  

HANA在硬盘上和内存里的数据存储方式是一样的  
以页的方式存储在硬盘上，大小4k-16M之间  

事务里面，每次写操作都有log产生才内存，但到commit的时候才写到磁盘上  
REDO/UNDO log保证完整性  
Redo log在写入数据之前（Savepoint）就被写入磁盘  

SAVEPOINT可以被手动保存为快照，直到手动删除  

Core Data service支持通用的数据包装，类似LINQ, 如Order.Customer.Address而不需要用JOIN来操作  
RDL新的HANA语言，运行于Core Data service之上,高级语言，类似JAVA  

CE中可以优化，例如没用用到的列会被移除，但是SQL语句不会  

Name server在每个host都有且缓存了数据，方便快速查询,但是只有master name server保存数据在磁盘上  

HANA可以重新组织表的分布，根据统计把经常join的表移动到一个节点上，移动的时候为了保持效率，先创建一个link到远程磁盘，等被使用的时候在load数据到内存  
表分区的几个好处：  
1.超过20亿行的大表  
2.并行在partition上  
3.query pruning，即可以不扫描不需要的分区  

hash/range如果有key的话必须是key的一部分  
round robin不能有key  

可以将一些表复制到其他的index server上，以提高性能  

建议是少于5亿行的表不需要分区,分区的话一般是3亿一个表  
分区会增加内存,因为每个分区都有自己的字典,不共享的  
分区表最好不要有unique约束,因为要到各个分区检查,会有性能问题  
2个表Join的时候分区建议是用相同的字段和分区数  
每个表分区的限制是1000个  

可以限制查询在某个分区上  
SELECT * FROM mytab WITH PARAMETERS ('RESTRICT_PARTS' = ('SYSTEM:MYTAB', '3', '4'))  

odbc/jdbc可以做load balancing功能，初次连接的时候获得系统信息，再连接的时候选择负载比较低的host, 以后每次都会更新负载信息，在长连接的时候可能会自动换到不同host，连接过程中如果一个host坏了那么自动切到另一个上  

在执行查询的时候，prepare statement时候获取信息（如表在哪个host上），执行statement的时候直接把request发到那个host上  

在插入大量数据的时候(bulk insert)，client会自动拆分到不同host上执行以提高性能  

Flexible Table:  
`CREATE COLUMN TABLE <table name> ( <column definitions> ) WITH SCHEMA FLEXIBILITY`  
Key-value类型的表,一行是一个对象,每列为Key, 值为value(s)  
在插入时候如果列不存在则建立一个  

HA:
HANA里面HA为  
1.硬件方面，为vendor提供，对HANA透明，如raid,网络等  
2.Watchdog，自动启动服务  
3.存储和备份到硬盘  
4.Standby/Failover, 分为N+m和N+N2中，其中N+N可以选择log被写入远程硬盘，或远程内存，或不等待写入3中情况  

认证和授权  
HANA支持3中方式，用户名密码，Kerberos(网络认证), SAML（HTTP）  

授权可以做到某用户看到某列（col>200），在model view上可以看到某年的数据等  

在XS里，现在可以支持SAP logon ticket和user/pwd方式认证  
2此请求之间不能保存数据  

Delivery Unit是几个package的集合  

HANA支持模糊搜索，同义字，自动补全等  

一个系统上多个隔离的数据库（云端，multi-tancy?）在设计阶段，没有实现  

L1（32K）和L2(256K)是在Core上面，L3(30M)是在Core之间  

缺省HANA会保留90%的系统内存供使用，但是可以在global.ini->global_allocarion_limit里配置(0为缺省)  

在系统在启动或者还没运行sql时候，可以通过Open dianogist mode来看系统状态  

Caculation View不选多维报告的话可以没有measure.   

HANA里的权限：  
SYSTEM privilege: 加减用户，创建数据库，备份等  
Object privilege: 创建修改表，视图等  
Analytic privilege: 查询时候读取分析视图的权限  
Package Privilege: 设计时修改创建各种分析模型的权限  

XS Engine:  
XS可以视为一个特殊的index server,去掉了数据库部分, XS可以和index server在不同进程中运行（缺省），也可以host在一个index server中运行（这样就不用跨进程取数据）  
.xsaccess用于认证，访问那些资源需要认证  

ODATA可以用Calucalated Views来实现读逻辑（而不是简单的从表读）  
XSCFGM定义数据结构，XSCFGD存前面定义结构的实际值，这2个是配置文件  

xs可以有定时或者trigger的job, 而不用前台触发:xsJob( param )  

匿名链接，.xssqlcc anonymous_connection, 在没有用户登录的时候用这个连接到HANA  
可以在xssqlcc里定义连接细节，用不同于登录用户的user连接到HANA执行SQL  
