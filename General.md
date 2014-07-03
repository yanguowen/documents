# 通用

### Web
JSONP:把JSON数据包在一个javascript方法里,可以跨源调用,因为浏览器允许跨源请求&ltscript&gt资源.

Cookie不可跨域名,即google只能访问google的cookie,不能访问baidu的cookie
同一个域下2个二级域名共享Cookie。可以设置domain

Session也是使用cookie来保持数据,存储了JSESSIONID
如果浏览器不支持cookie, 则可以使用url重写,把session的id添加在url后面用作验证, encodeURL会自动判断是不是支持cookie, 不支持则重写url

Session劫持：取得客户端的cookie里面的sessionId, 然后在别的机器上发起会话，劫持到别人的session
防范1：设置cookie为httponly, 那么客户端脚本不能访问此cookie
防范2：每个请求带一个动态token, 每次验证token.

Session:
1. Cookie存key, 服务器存value(可以放在内存或者数据库里)
2. Cookie存加密过后的value，则服务器不需要存任何信息，只要解密即可
在分布式的情况下，如果放在内存里，则前端反向代理服务器随机选择一个服务器来连接的话，可能会失去之前的session信息(可以采取算法设置同一个来源都映射到同一个服务器应答)
采用session共享，如tomcat, 把session复制到其他tomcat行，性能会很有问题。
如果放在数据库里和普通表放在一起，则数据库集群的话也需要同步，可以防止一个专门存放session的数据库里
还有就是用memcache同步session(会产生内存碎片)

Web分布式系统:
1. 单机  
2. 应用和数据库分离  
3. Squid做反向代理，缓存静态页面  
4. 页面片段缓存  
   a. CSI(iframe, javascript等方式将另外一个页面包含进来)  
   b. SSI,通过注释SSI命令加载不同模块，需要服务器支持，如<!--#include virtual="header.html"-->, 不能跨域  
   c. ESI(在缓存服务器上通过标记来说明哪些可以缓存哪些不行)，如<esi:include src="login.php" />  
5. 数据缓存。把常用的数据缓存在内存里，如memcache等  
6. 增加服务器，如2个tomcat，需要负载均衡，session同步，分布式缓存等  
7. 数据库集群或者分库（比如把没有关联的表放到不同的服务器上面）  
8. 分表，data access layer层（Sharding）  
   生成全局主键:  
   UUID, 缺点是很长，占用存储空间，并且索引存在性能问题  
   维护一个表记录每个表的笑一个主键是什么，有单点问题  
   Sharding有可以在几个层次：DAO层（自己实现），ORM层（框架），JDBC层(技术难度高)，Appserver和db之间的代理实现(Mysql proxy)  
   保持事务性:分布式事务（2阶段提交）,Best Efforts   1PC模式,事务补偿机制（最终一致性，用逻辑来检查，不要求实时，如用日志来对账等等）  
   跨节点Join问题，2次查询，第一次找出相关id,然后到别的上面查询。  
   聚集函数问题（因为要在全数据上面统计），在各个节点上的到结果然后到应用层合并  
9. 大量的web server,硬件负载均衡，分布式文件系统等  
10. 数据库读写分离，bigtable等等  
11. 通用框架，如google,taobao等  


跨站请求伪造(Cross-Site Request Forgery):意思是比如用户登录了招商银行网站，还没有退出的同时（或者cookie还有效），访问了另一个网站，这个网站上有个连接点了以后会去招行转账。

OAUTH: 用户在访问B网站的时候要用到A网站的东西，这时候B会把用户导向A的登录授权页面，然后自动返回

消息中间件: 通过消息中间件，应用程序或组件之间可以进行可靠的异步通讯来降低系统之间的耦合度，从而提高整个系统的可扩展性和可用性。解耦，异步和并行
达到最终结果一致
主要用来处理一些不需要适时的应用，如淘宝付款成功产生消息，后面仓储、分析等消费消息，还有数据复制等应用


CDN:
视频网站百万人在线带宽为T级别
每台机器大概1000链接，1G带宽
1000台以上服务器,10分钟分发完毕
请求合并：100个人请求相同内容，可以合并为一个请求
smokeping，测试2个节点间网络带宽，丢包等

#### RESTful
为所有“事物”定义ID
将所有事物链接在一起--使用uri来标定对象,而不是内部id, 这样在哪里都可以访问
使用标准方法:get/post/put等
资源多重表述:就是返回多一些格式如xml, json等
无状态通信,服务器端不保持状态,客户端的调用不依赖服务器,容易扩展,也有很好的性能

RESTFul authentication:
1. HTTP basic auth, 第一次会跳出一个登录按钮（不可定制），以后就不用
2. Session，可以再服务器端和客户端(Cookie)，都不是严格无状态的
3. API query string签名

#### Node.js
单次加载, Node中require模块只会被加载一次,只有一个实例

模块是一个文件,包是一个目录

本地模式安装在nodejs目录下,可以reqire, 没有加入PATH, 全局模式安装在系统目录,不可以require, 加入path, 可以在命令行运行

### 一般
线程有自己的PC, 寄存器，栈和帧等

Currying 克利化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术
如，f(x,y) = x + Y, 如果传递f(2)则返回函数f(y) = 2 + y, 闭包利克利化来实现

尾递归:tail recursive，就是从递归的最后条件开始算，直接可以计算出值返回，而且出现在函数最尾部，所以不需要保存任何局部变量。一般需要额外的参数来传递结果值

跳表：有几层，每层都是链表，上层相当于索引

### 大数据
Hadoop:  
  namenode是HDFS的元数据信息，都存在内存里，一个文件大概要150个字节，所以过多的小文件不适合在HDFS里  
  datanode为工作节点，类似于nameserver和indexserver  
  HDFS里只有一个writer，不支持多个同时写入，写总是append到文件末尾，原则是一次写入，多次读取分析。  
  HDFS中块的大小为64M, 所以比较适合大文件,Hadoop的map主要是以块为处理对象，而不是文件。  
  Combine函数式用来合并本地数据的，一个map处理完了以后可以用combine函数在本地合并，比如将很多the, 1合并为the, 999然后传递给reduce  

Pig:  
  Pig是基于Hadoop上的一个高层应用，有自己的语法，可以写出脚本的语句来解决问题，主要就是不需要复杂的Hadoop编程, 不同于SQL, Pig是基于数据流的,主要的操作是变形，但是SQL是描述型的
  
Hive:  
  是基于Hadoop上的数据仓库框架，用类似SQL语句而不是Java语言来组织,Hive吧SQL查询转换为一系列集群上运行的MapReduce作业,Hive适合离线操作  
  Hive把数据组织委表，所以需要create table,有些语句可以直接把数据从文件读到表里  
  Hive是读时模式(Schema on read),一般数据库是写时模式(Schema on write)，意思是一般的数据库是在写入的时候检查schema表，约束等，但Hive是在读时候，所以写入速度很快，基本就是copy文件
  之所以会有这个需求是因为很多数据时固定的，但是查询时未知的，所以没办法在建表的时候知道该建立是么索引  
  
HBase:  
  在HDFS上面向列的分布式数据库,支持实时随机读取。  
  
ZooKeeper是Hadoop的分布式协调服务  

Sqoop是一个工具，可以把数据源从RDBMS中抽取到Hadoop中，类似于SLT/DS的东东  


