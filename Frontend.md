# 前端技术

## Javascript

var定义的是作用域里面的局部变量  
不用var在任何地方定义的都是全局变量  
Object其实是一组key-value的集合  
=== 和 !==不对操作数转换,只有类型相等并且值相等才相等, == !=会类型转换  

JS只有函数式域,别的都不是,所以在大括号里面定义的东西到外面也可以用

函数的参数实际是一个数组,所以可以传任何数量的参数,定义一个可以实际传三个,定义三个也可以传一个,签名不做检查,所以函数也不能重载函数实际上是对象,函数名则是一个指针,有自己的属性和方法,里面的this随着调用对象的不同而不同  
函数的apply和call用来显示调用函数,如`sum.apply(this, argc)`和`sum.call(this, num1,num2)`, 主要的目的是为了改变函数的调用域,另外也可以用bind函数将其绑定到一个对象上

js里面没有块作用域,在{}里面定义的变量外面也可以用
`person.name`与`person["name"]`是一样的,[]的有点在于可以用变量来访问属性,如:
`var pname = "name";`  
`person[pname];`  

数组可以用push/pop来作为栈操作,也可以用push/shift作为队列操作
concat添加到后面,没有参数则复制一个,slice切片,相当于substr, splice用于删除或者替换,返回的是删除的项
every/filter/forEach/map/some迭代方法,对数组里的每一项run函数
reduce用于归并,如实现sum

RegExp正则表达式,/pattern/flags, var pat = /at/g g:全局,i:不区分大小写,m:多行

Global的eval方法本质上是把string里的script插入当前位置执行,所以可以有

Prototype是一个指针,指向一个公共的地方(类似于父类的实例,所有的子类对象都是共享的)
用Prototype里的方法比在构造函数里定义性能要好,因为构造函数每次实例化都会执行

JS的闭包就是一个外部的指向函数内的函数的指针,可以保持函数内的变量不会被回收.

JS的继承,注意，Sub 仅仅继承了 Base 在原型中定义的函数，而构造函数内部创造的 base 属性和 sayHello 函数都没有被 Sub 继承

