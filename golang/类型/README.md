# golang 类型

## 变量

### 命令名方式

> 使用var定义变量，支持类型推断，也可直接进行初始化，也可以省略var关键字

- 变量定义类型，再赋值

- 直接复制（包含var，不包含var）不包含var定义变量复制，必须采用`:=`
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

