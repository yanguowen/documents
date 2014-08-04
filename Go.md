# Go语言

### 语法
在for循环中，因为go不支持逗号运算符，所以不能类似c语言的`i=i+1,j++`这样的,必须使用平行赋值，如`i,j=i+1,j+1`
obj.(Type) 或者 obj.(interface)可以实现接口或者类型查询,返回接口或者类型名  
每个文件都可以有一个不带参数的init函数，由系统自动调用，在程序包中所有变量声明都被初始化，以及所有被导入的程序包中的变量初始化之后才被调用  
值方法可以在指针和值上进行调用，而指针方法只能在指针上调用,因为指针方法可以修改数据，值方法不行。  
字符串：Go中的字符串是一个byte的数组，而不是字符(char)的数组，所以如果用index的话得到的是byte的列表  
go的源代码是存为UTF-8格式的，所以所有的字符串常量在保存的时候已经以UTF-8格式存好了。
go中的rune是Unicode中的代码点，如\u1234

### 定义域赋值
:= 在同一层代码中可以被重复赋值，而不会定义出来。在不同代码层中会定义新的变了，如:
```golang
    s := "hi"  
    {  
        s := "go"  //新的变量  
    }
``` 
### 引用类型
包含三种，slice, map和channel
new用来分配值类型数据，初始化为零值，分配内存并返回指针，make用来创建三种引用类型，返回实例  
new和make都是函数而不是操作符

### 类型检查
p.(type) 返回p的真正类型，只能在switch里面调用  
也可以用something.(I)来测试something是不是实现了I接口

### 指针
返回局部变量指针是安全的，程序会将其分配在GCHeap上，如下:
```golang
func test() *int {  
    x := 100  
    return &x  
}  
```
### Range 
类似于迭代器，返回Key,Value  
注意，range 会赋值对象,所以改变value是没用的。要用指针或者下标访问
对于slice, range会访问到len而不是cap, 

### 可变参数
可变参数本质就是一个slice, 必须在最后一个  
在使用slice作为参数传递时，必须展开  
```
func main() {
    s := []int{1, 2, 3}
    println(test("sum: %d", s...))
}
```
### 错误处理
panic（）抛出错误，recover（）捕获错误, panic会一直持续向上，知道recover,recover只能在defer函数中调用, recover在正常的情况下会没有作用，返回nil

### Array
Array是值类型，传递参数和赋值会复制整个数组，而不是指针
支持多维数组，第二维不能可变
`b := [...][2]int{{1, 1}, {2, 2}, {3, 3}}`
这个是数组
`arr := [...] int {1,2}`
这个是slice
`slc := []int{1,2}`

### Slice
Slice是引用类型，但是本身是结构体  
可以直接创建slice,底层数据自动创建（Make）  
reslice是基于已有slice来创建新的slice  
append操作如果没有超出（cap）底层数组，那么会覆盖后续数据，如果超出cap, 会分配新的空间
在用下标访问slice时候不能超出len
copy会复制，按照2个slice中小的一个来计算

### Map
引用类型，键值必须支持==和!=操作  
从map中取回的是value的复制品，对他操作没有意义，建议使用指针  

### Struct
值类型，可以在字段后加标签，用于反射读取，如：  
```
var u1 struct { 
	name string "username" 
}
```
### 接口
空接口相当于一般语言的Object或者是void*  
```
var v1 interface{} = 1 // 将int类型赋值给interface{}
var v2 interface{} = "abc" // 将string类型赋值给interface{}
var v3 interface{} = &v2 // 将*interface{}类型赋值给interface{}
var v4 interface{} = struct{ X int }{1}
var v5 interface{} = &struct{ X int }{1}
```
给接口赋值会导致拷贝临时变量保存在接口里，只能读不能修改。如下:  
```
type User struct {
    id int
    name string
}
func main() {
    u := User{1, "Tom"}
    var vi, pi interface{} = u, &u
    // vi.(User).name = "Jack" // Error: cannot assign to vi.(User).name
    pi.(*User).name = "Jack"
    fmt.Printf("%v\n", vi.(User))
    fmt.Printf("%v\n", pi.(*User))
}
```
输出：
```
{1 Tom}
&{1 Jack}
```
让编译器检查，以确保某个类型实现接口。  
`var _ fmt.Stringer = (*Data)(nil)`

### 并行
调度器不能保证多个 goroutine 执行次序，且进程退出时不会等待它们结束。  
默认系统只用一个thread来跑所有的goroutine, 可以设置runtime.GOMAXPROCS来实现真正的并行
goroute默认会一直占用CPU知道结束，所有需要调用Gosched来让出CPU给别的Goroutine

Channel为并发安全的，默认为同步模式，会阻塞，异步方式通过判断缓冲区是否满或者空来阻塞  
chan发送方也会阻塞，在没有读取的时候会一直阻塞
由于在Go中，数据同步发生在从channel接收数据阶段  
强制转换为单向Channel，但是单向不能转为双向，单向chan只是在传递给函数时候使用，防止错处，在调用出还是双向的  


```
c := make(chan int, 3)
var send chan<- int = c // send-only
var recv <-chan int = c // receive-only
```
Select随机选择包含的channel或default, 如果没有则阻塞,select里面的case必须是IO操作
`select {}`
会阻塞goroutine

可以利用自己读写chan来实现代码段的lock，如下:  
```
sem <- 1
for x := 0; x < 3; x++ {
    fmt.Println(id, x)
}
<-sem
```
利用select实现超时处理   
```
select {
	case v := <-c: 
		fmt.Println(v)
	case <-time.After(time.Second * 3): 
		fmt.Println("timeout.")
}
```
多个函数从一个chan读取数据称为fan out,一个函数从多个chan读称为fan in.

### 锁
sync.Mutex和sync.RWMutex,前一个是一般锁，后一个是单写多读锁，用Lock为写Lock, RLock为读lock
sync.Once有一个once.Do(func)的分发，可以保证在运行期只调用一次的作用

### 网络
go支持rpc调用，服务器端注册rpc.Register, 客户端既调用,网络传递数据用Gob格式，为而二进制流，所以不能跨语言使用  
json.Marshal(obj) 可以序列化一个对象为json格式，byte数组。  
Gob是用名字来对应的，所以如果接收方的结构没有那个成员则忽略，如果是指针也可以对应上

### 工具
gofix是一个工具，在api升级的时候可以自动把老的api转为新的，如果不能自动转换则给出警告，要人工改写


