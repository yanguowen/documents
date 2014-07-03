# Go语言

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
主语，range 会赋值对象。

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
panic（）抛出错误，recover（）捕获错误

### Array
Array是值类型，传递参数和赋值会复制整个数组，而不是指针
支持多维数组，第二维不能可变
`b := [...][2]int{{1, 1}, {2, 2}, {3, 3}}`

### Slice
Slice是引用类型，但是本身是结构体  
可以直接创建slice,底层数据自动创建（Make）  
reslice是基于已有slice来创建新的slice  
append操作如果没有超出（cap）底层数组，那么会覆盖后续数据，如果超出cap, 会分配新的空间

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



