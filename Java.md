# Java
## Web
Servlet在初始的时候启动,以后每次来一个请求则创建一个新线程执行service 方法
Servlet不是线程安全的
@PostContructor和@PreDestory可以在销毁和生成的时候执行额外的函数
注解会影响启动速度
Forward在不同servlet间跳转

