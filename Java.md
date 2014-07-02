# Java
## Web
Servlet在初始的时候启动,以后每次来一个请求则创建一个新线程执行service 方法  
Servlet不是线程安全的  
@PostContructor和@PreDestory可以在销毁和生成的时候执行额外的函数  
注解会影响启动速度  
Forward在不同servlet间跳转  

请求到servlet之前经过一个filterChain, 安装filter则可以做很多过滤操作,如权限认证,数据压缩,字符串编码等
Listener监听器,可以在某些事件发生的时候出发动作,如session的变化

类加载器层级结构，一般先用父类加载器加载，但在web server如tomcat中，为每个web app提供一个独立的类加载器，优先使用此加载器
层次结构和代理模式，代理给自己的父类

Thread.run和构造函数中实现Runnable接口的对象

## 编码
锁:synchronized 互斥
同步: wait/notify/notifyAll

