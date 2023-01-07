# golang 类型

## 变量

### 命令名方式

使用var定义变量，支持类型推断，也可直接进行初始化，也可以省略var关键字

- 变量定义类型，再赋值

- 直接复制（包含var，不包含var）不包含var定义变量复制，必须采用`:=`

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
}
```

### 表达式

if

switch

for

### 函数
