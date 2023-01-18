# 2.golang 类型

## 2.1.变量

### 变量定义

> 关键字用于`var`用于变量定义，类型放在变量后，变量初始化值可省略变量类型，由编辑器自动推断

```go
var x  int //自动初始化为0
var x=1 //自动推断为int类型
```

> 可以一次性定义多个变量，包括用不同的初始值定义不同的类型

```go
var x,y int //自动初始x,y，并赋值0
var a,s=100,"abc" //初始化a为100整型变量，s为abc的字符变量
```

> 通常写法，以组方式进行变量定义

```go
var (
    x,y int
    a,s=100,"abc"
)
```

### 简短模式

> 除var关键字外，还可以使用更加简短的变量定义和初始化语法
>
> **简短模式由一定的限制**
>
> - **定义变量同时显示初始化（必须设置变量值）**
> - **不能提供数据类型**
> - **智能用在函数内部，函数外不能采用简短模式的定义方式**
>
> 简短模式在函数多返回值，`if/for/switch`等语句中定义局部变量非常方便
>
> 简短模式用于变量定义，也可用于退化赋值操作（退化赋值需要在同样一个作用域中，如`for，switch、if`，或者函数体中）

```
package main
import(
	"log"
	"os"
)
func main(){
	f,err:=os.Open("/dev/random")
	...
	buf:=make([]byte,1024)
	n,err:=f.Read(buf)  //n为定义，并且赋值，而err则是退化赋值
	...
}
```

## 2.2.命名

- 字母或下划线开始，由多个字母数字和下划线组合而成
- 区分大小写
- 使用驼峰拼写格式
- 局部变量优先使用短名
- 不要使用保留关键字
- 不建议使用预定义常量、类型、内置函数相同的名字
- 专用名词通常会全部大写，比如DBUtil，escapeHTML

> 命令首字母大小写决定了其作用域，类似java的public和private的作用，这点需要注意
>
> 多参数接收部分参数不使用情况和和包引入未使用的情况可使用空标识符`_`

```go
import(
    "fmt"
    _ "gitee.com/yaohao803/util" //util包该类未使用
)
x,_:=strconv.Atoi("12") //忽略在转换时的错误

```

## 2.3.常量

> 用于代替魔法数
>
> **常量不能使用简短模式赋值**
>
> 在程序中不曾使用的常量不会引发编译错误

```go
const x,y int =123,0x22

const(
	i,f =1,0.123 //分组方式赋值
)
```

> 在常量组中如不制定类型和初始化值，则与上一行非空常量右值相同

```go
func main(){
	const(
		x uint16=120
		y  //与x的类型、右值相同
		s="abc"
		z //与s类型、右值相同
	)
}
```

> go中没有枚举值，可借助iota标识符实现自增常量，实现枚举类型.iota为常量计数器，智能在常量中使用

```go
const(
	x = iota //0
	y //1
	z //2
)

const(
    _ = iota //0
    KB = 1<<(10*1) //1<<(10*1)
    MB             //1<<(10*2)
    GB             //1<<(10*3)
)
```

> iota可以在常量中定义多个，他们各自单独计数，只需要保证每行常量的列数量相同即可

```go
const(
	_,_=iota,iota*10 //1,0*10
	a,b  //1,1*10
	c,d  //2,2*10
)
```

> 如果iota终端，需要显示回复才会恢复取值

```go
const(
	a=iota //0
	b //1
	c=100 //100
	d //100(与上一行常量右值表达式相同)
	e=iota //4(恢复iota计数，计数包括前面的c，d)
	f //5
)
```

## 2.4.基本类型



- | 类型          | 长度 | 默认值 | 说明                             |
  | ------------- | ---- | ------ | -------------------------------- |
  | bool          | 1    | false  |                                  |
  | byte          | 1    |        |                                  |
  | int，uint     | 4，8 |        |                                  |
  | int8，uint8   | 1    |        |                                  |
  | int16，uint16 | 2    |        |                                  |
  | int32，uint32 | 4    |        |                                  |
  | int64，uint64 | 8    |        |                                  |
  | float32       | 4    |        |                                  |
  | float64       | 8    |        |                                  |
  | complex64     | 8    |        |                                  |
  | complex128    | 16   |        |                                  |
  | rune          | 4    |        |                                  |
  | uintptr       | 4，8 | 0      | 足以存储指针的uint               |
  | string        |      | ""     | 字符串，默认为空字符串，而非null |
  | array         |      |        | 数组                             |
  | struct        |      |        | 结构体                           |
  | function      |      | nil    | 函数                             |
  | interface     |      | nil    | 接口                             |
  | map           |      | nil    | 字典，引用类型                   |
  | slice         |      | nil    | 切片，引用类型                   |
  | channel       |      | nil    | 通道，引用类型                   |

  变量定义类型，再赋值

- 直接赋值（包含var，不包含var）不包含var定义变量复制，必须采用`:=`
- 变量定义赋值放在同一行一起进行定义，赋值，如`a,b,c:=10,5,"hello world"`
- `=`是赋值语句，`:=`是声明+赋值

```go
package main

func main() {
	var a int32
	var b string
	a = 12
	b = "hello world1"
	println(a, b)

	var c = 12
	var d = "hello world1"
	println(c, d)

	e := 13
	f := "hello world3"
	println(e, f)
    
    g,h,i:=10,5,"hello world4"
    println(g,h,i)
}
```

### 表达式

> 支持三种流控语句，与其他语言相比较简单

#### if

> 注意，if，else和java不同，不能带小括号

```go
func s0102() {
	x := 100
	if x > 0 {
		println("x")
	} else if x < 0 {
		println("-x")
	} else {
		println("0")
	}
}
```



#### switch

```go
func s0103() {
	x := 100
	switch {
	case x > 0:
		println("x")
	case x < 0:
		println("-x")
	default:
		println("0")
	}
}
```



#### for

> 在go中，没有while语句，通过for，可以实现while的功能
>
> go语言的for条件不要带括号

```go
func s0104() {
	//基础用法
	for i := 0; i < 5; i++ {
		println(i)
	}

	//替代while
	x := 0
	for x < 5 {
		println(x)
		x++
	}

	//替代while(true)
	x = 4
	for {
		println(x)
		x--
		if x < 0 {
			break
		}
	}
}

```

> 在进行迭代遍历时，如果需要返回索引，则使用for...range

```go
func s0104() {
	y := []int{100, 200, 300}
	//i 是索引，v是遍历数组中的对象
	for i, v := range y {
		println(i, ":", v)
	}
}
```

### 函数

> 函数可定义多个返回值，还可对于返回值进行命名

```go
// 函数定义
func div(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("除数不能为零")
	}
	return a / b, nil
}
```

函数调用接收参数需要和函数返回参数保持一致，接收的参数如果没有再当前程序中有用，则可以用`_`表示

```go
// 函数调用
func s0105() {
	a, b := 10, 2
	c, err := div(a, b)
	fmt.Println(c, err)
	b = 0
	c, err = div(a, b)
	fmt.Println(c, err)
	d, _ := div(a, b)
	fmt.Println(d)
}
输出：
5 <nil>
0 除数不能为零
0
```

