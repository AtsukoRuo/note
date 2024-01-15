# Go 初步



Go语言的演化主要分为三个分支

- 从C语言中，Go继承了表达式、控制流语句、基本数据类型、指针等等语法。而且它还继承了C语言所要强调的要点，即程序需要编译成高效的机器码，并且自然地与所处的操作系统提供的抽象机制相配合。
- 从Module-2、Oberon等语言借鉴了包、模块化地概念
- Go also is inspired by the concept of **communicating sequential processes (CSP)** from TonyHoare’s seminal 1978 paper on the found ations of concurrency. In CSP,a program is a parallel composition of processes that have no shared state; the processes communicate and synchronize using channels.

![image-20231224150022801](C:\Users\AtsukoRuo\AppData\Roaming\Typora\typora-user-images\image-20231224150022801.png)





Go是编译型的语言，它的工具链将程序的源文件转变成机器相关的原生二进制指令。

Go对于代码格式有着十分苛刻的要求，以下情况都会编译失败：

- 缺失导入或者存在不需要的包
- 声明但未使用的变量
- 左大括号与某些语句（for、func）不在同一行。



## 命名

命名规则：

1. 以字符或者下划线开头（支持Unicode）

2. 数字、字符、下划线（支持Unicode）

3. 对大小写敏感

4. 关键字在大部分上下文中不能作为名字

   ~~~go
   break 		default 	func 	interface 	select
   case 		defer 		go 		map 		struct
   chan 		else 		goto 	package 	switch
   const 		fallthrough if 		range 		type
   continue 	for 		import 	return 		var
   ~~~

命名风格是驼峰式的。

   

预声明的常量、函数、类型不是保留的，你可能在声明中使用它们，但是会有潜在的命名冲突问题。

   ~~~go
   true false iota nil
   ~~~

   ~~~go
   int int8 int16 int32 int64
   uint uint8 uint16 uint32 uint64 uintptr
   float32 float64 complex128 complex64
   bool byte rune string error
   ~~~

   ~~~go
   make len cap new append copy close delete
   complex real imag
   panic recover
   ~~~



在函数内声明的变量，仅在函数局部作用域中可见。在函数外声明的变量，对本包中所有其他的源文件可见。

如果第一个字母是大写的，那么它在包外也是可见的（导出的），否则仅在包内可见。

> 对于中文汉字，Unicode标志都作为小写字母处理，因此中文的命名默认不能导出。不过国内的用户针对该问题提出了不同的看法，根据RobPike的回复 [Issue763](https://github.com/golang/go/issues/5763) ，在Go2中有可能会将中日韩等字符当作大写字母处理

## 声明

声明是给一个实体命名，并且指定该实体的性质。Go语言主要有四种类型的声明语句：`var`、`const`、`type`和`func`，分别对应变量、常量、类型和函数实体对象的声明。

## 变量

var声明语句可以创建一个特定类型的变量，语法如下

~~~go
var 变量名字 类型 = 表达式
~~~

初始化表达式可以是字面量或任意的表达式。在包级别声明的变量会在main入口函数执行前完成初始化，局部变量将在声明语句被执行到的时候完成初始化。



如果省略类型，那么将根据初始化表达式来推导变量的类型。

如果省略表达式，那么用零值初始化该变量。零值初始化机制可以确保每个声明的变量总是有一个良好定义的值：

- 数值类型变量对应的零值是0
- 布尔类型变量对应的零值是false
- 字符串类型对应的零值是空字符串
- 接口或引用类型（包括slice、指针、map、chan和函数）变量对应的零值是nil
- 数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。



也可以在一个声明语句中同时声明一组变量

~~~go
var i, j, k int
var b, f, s = true, 2.3, "four"
~~~



在函数内部，有一种简短变量声明语句的形式，可用于声明和初始化局部变量，语法如下：

~~~go
名字 := 表达式
~~~

变量的类型根据表达式来自动推导。

~~~go
anim := gif.GIF{LoopCount: nframes}
freq := rand.Float64() * 3.0
t := 0.0
i, j := 0, 1 	//声明和初始化一组变量
~~~

var形式的声明语句往往是用于

- 需要显式指定变量类型的地方
- 变量稍后会被重新赋值，并且初始值无关紧要的地方。



**请记住“:=”是一个变量声明语句，而“=”是一个变量赋值操作。**这里有一个比较微妙的地方：简短变量声明左边的有一些变量，已经在**相同的作用域**声明过了，此时对于已经声明过的变量就只有赋值行为了。

~~~go
in, err := os.Open(infile)
// ...
out, err := os.Create(outfile)
~~~

简短变量声明语句中必须至少要声明一个新的变量，下面的代码将不能编译通过：

~~~go
f, err := os.Open(infile)
// ...
f, err := os.Create(outfile) // compile error: no new variables
~~~

解决的方法是第二个简短变量声明语句改用普通的多重赋值语句。



### 指针

Go还支持指针的特性。在Go语言中，返回函数中局部变量的地址也是安全的。

~~~go
var p = f()

func f() *int {
	v := 1
	return &v
}
~~~

在 Go 语言中，如果可能，编译器会在函数的栈帧中为局部变量分配空间。然而，如果编译器确定某个变量必须在函数返回后继续存在，那么该变量将被分配在堆上，用Go语言的术语说，这个x局部变量从函数f中逃逸了。

~~~go
fmt.Println(f() == f()) // "false"
~~~



另一个创建变量的方法是调用内建的new函数。表达式new(T)将创建一个T类型的匿名变量，初始化为T类型的零值，然后返回指针`*T`。

~~~go
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
~~~

每次调用new函数都是返回一个新的变量的地址，因此下面两个地址是不同的：

```
p := new(int)
q := new(int)
fmt.Println(p == q) // "false"
```

当然也可能有特殊情况：如果两个类型都是空的，也就是说类型的大小是0，例如`struct{}`和`[0]int`，那么有可能有相同的地址（依赖具体的语言实现）



### 生命周期

变量的生命周期指的是在程序运行期间变量有效存在的时间段。

- 对于在包一级声明的变量来说，它们的生命周期和整个程序的运行周期是一致的。

- 局部变量的生命周期则是由Go语言的自动垃圾收集器来负责。

  函数的参数变量和返回值变量都是局部变量。它们在函数每次被调用的时候创建。



### 赋值

~~~go
x = 1                       // 命名变量的赋值
*p = true                   // 通过指针间接赋值
person.name = "bob"         // 结构体字段赋值
count[x] = count[x] * scale // 数组、slice或map的元素赋值
~~~



元组赋值是另一种形式的赋值语句，它允许同时更新多个变量的值。在赋值之前，赋值语句右边的所有表达式将会先进行求值，然后再统一更新左边对应变量的值。

~~~go
x, y = y, x

a[i], a[j] = a[j], a[i]
~~~

但如果表达式太复杂的话，应该尽量避免过度使用元组赋值；因为每个变量单独赋值语句的写法可读性会更好。

我们可以用下划线空白标识符`_`来丢弃不需要的值。

~~~go
_, err = io.Copy(dst, src) // 丢弃字节数
_, ok = x.(T)              // 只检测类型，忽略具体值
~~~



### 类型

变量或表达式的类型定义了其属性特征，例如数值在内存的存储大小，它们在内部是如何表达的，是否支持一些操作符，以及它们自己关联的方法集等。

而类型的语义是由程序员自己来解释的。例如，一个int类型的变量可以用来表示一个循环的迭代索引、或者一个时间戳、或者一个文件描述符、或者一个月份；一个float64类型的变量可以用来表示每秒移动几米的速度、或者是不同温度单位下的温度；一个字符串可以用来表示一个密码或者一个颜色的名称。

一个类型声明语句创建了一个新的类型，和现有类型具有相同的底层结构，**并且提供了更加丰富的语义信息**。**但是新的类型和原类型是不兼容的，即使它们具有相同的底层结构。**

~~~go
type 新类型名 旧类型名
~~~



新类型还可以定义额外的行为，这些行为表示为一组关联到该类型的函数集合，我们称为类型的方法集。我们将在第六章中讨论方法的细节，这里只说些简单用法。

~~~go
func (c Celsius) String() string { return fmt.Sprintf("%g°C", c) }

// 使用
fmt.Println(c.String()) // "100°C"
~~~

### 类型转换

在 Go 语言中，不存在隐式类型转换。

类型转换通常不会改变值本身，但是一定会使它们的语义发生变化。对于每一个类型T，都有一个对应的类型转换操作T(x)，用于将x转为T类型。只有当两个类型的底层基础类型相同时，或者是两者都是指向相同底层结构的指针类型，才允许这种转型操作。



数值类型之间的显式转型也是允许的，并且在字符串和一些特定类型的slice之间也是可以转换的。这类转换可能有改变值的行为，例如：

1. 将一个大尺寸的整数类型转为一个小尺寸的整数类型
2. 将一个浮点数转为整数



### 包

Go语言中的包和其他语言的库或模块的概念类似，通常一个包所在目录路径的后缀是包的导入路径；例如包`helloworld`对应的目录路径是`$GOPATH/src/helloworld` 或者`$PROJECT_PATH/helloworld`。

**在Java中，强制要求包名要与目录名同名。而在Go中，包路径和包名是分离的。**

每个源文件的第一条语句（除注释）都必须是包声明语句`package`，用于指明包名。在Go语言中，包名的最佳实践是**包路径的最后一部分**。包名可以不同于目录名，然而并不推荐这种做法。

一个代码包与一个目录（不包括子目录）是一一对应的：

- 一个代码包可以由若干Go源文件组成，一个代码包的源文件须都处于同一个目录下。
- 一个目录下的所有源文件必须都处于同一个代码包中，亦即这些源文件开头的`package pkgname`语句必须一致。

对于Go官方工具链来说，一个引入路径中包含有`internal`目录名的代码包被视为一个特殊的代码包。该代码包仅对子包以及直接父包可见。

每个 Go 应用程序都包含一个名为main的包，而且在main包中必须有且仅有一个main方法。执行 `go install <package>` 命令后，系统会尝试在指定的包目录里寻找带有 `main` 包声明的文件，然后将main方法作为程序的入口。

每个包都对应一个独立的名字空间。例如，在image包中的Decode函数和在unicode/utf16包中的 Decode函数是不同的。要在外部引用该函数，必须显式使用image.Decode或utf16.Decode形式访问。



在每个源文件的包声明前紧跟着的注释是包注释。一个包通常只有一个源文件有包注释（译注：如果有多个包注释，目前的文档工具会根据源文件名的先后顺序将它们链接为一个包注释）。如果包注释很大，通常会放到一个独立的doc.go文件中。



包的初始化过程：

1. 在一个程序启动时，每个包中总是在它所有依赖的包都加载完成之后才开始加载。而且每个被用到的包会被而且仅会被加载一次。Go不支持循环依赖

2. 在加载一个代码包的过程中，所有的声明在此包中的`init`函数将被串行调用并且仅调用执行一次。

   - 在同一个源文件中声明的`init`函数，将按从上到下的声明顺序被调用执行。
   - 在同一个包中的两个不同源文件中的`init`函数，Go语言白皮书推荐（但不强求）按照它们所处于的源文件的名称的词典序列来调用

   这个init只能由编译器在初始化时调用，程序员不得调用

   ~~~go
   package main
   
   import "fmt"
   
   func init() {
   	fmt.Println("hi,", bob)
   }
   
   func main() {
   	fmt.Println("bye")
   }
   
   func init() {
   	fmt.Println("hello,", smith)
   }
   
   func titledName(who string) string {
   	return "Mr. " + who
   }
   
   var bob, smith = titledName("Bob"), titledName("Smith")
   ~~~

   

3. 同时Go编译器会解决包级变量的初始化顺序问题

   ```go
   var a = b + c // a 第三个初始化, 为 3
   var b = f()   // b 第二个初始化, 为 2, 通过调用 f (依赖c)
   var c = 1     // c 第一个初始化, 为 1
   
   func f() int { return c + 1 }
   ```



一个引入声明语句的完整形式为：

~~~go
import importname "path_to_package"
~~~

- `importname`是可选的，它的默认值为被引入的包名（不是目录名）
- `path_to_package`是包所在的路径，并不是包名

### 作用域

注意区分「作用域」与「生命周期」。

- 作用域对应的是一个名字的可见性；它是一个编译时的属性
- 生命周期作用对象是一个位于内存中的变量；它是一个运行时的属性

例如，for循环语句的作用域包括循环体以及初始化部分。



同一个作用域中，禁止包括多个同名的声明。当编译器遇到一个名字引用时，它会对其定义进行查找，查找过程从最内层的词法域向全局的作用域进行。如果查找失败，则报告“未声明的名字”这样的错误。如果该名字在内部和外部的块分别声明过，则内部块的声明首先被找到。在这种情况下，内部声明屏蔽了外部同名的声明：

~~~go
func main() {
	x := "hello!"
	for i := 0; i < len(x); i++ {
		x := x[i]
		if x != '!' {
			x := x + 'A' - 'a'
			fmt.Printf("%c", x) // "HELLO" (one letter per iteration)
		}
	}
}
~~~

注意到`x:=x[i]`语句中，左右两边的`x`虽然同名，然而是不同的实体。原因在于，当编译器遇到一个名字引用时，它会按照操作（例如，函数调用、数组访问等等）所期望的定义进行查找。



## 基本类型

### 整型

Go语言同时提供了有符号和无符号类型的整数运算

- `int8`、`int16`、`int32`和`int64`
- `uint8`、`uint16`、`uint32`和`uint64`

此外，还有以下整型类型

- `int`、`uint`，它们的大小取决于机器的具体实现。
- `rune`，是和int32等价的类型，通常用于表示一个Unicode码点
- `byte`，是uint8类型的等价类型，通常用于表示二进制原始数据
- `uintptr`，用于存储指针值



即使需要非负数的语义，也不要使用无符号数。例如我们看一个经典的溢出Bug

~~~go
medals := []string{"gold", "silver", "bronze"}
for i := len(medals) - 1; i >= 0; i-- {
	fmt.Println(medals[i]) // "bronze", "silver", "gold"
}
~~~

出于避免溢出情况的考量，我们一般在操纵二进制数据时才使用无符号类型。



### 浮点数

Go语言提供了两种精度的浮点数，float32和float64。它们的算术规范由IEEE754浮点数国际标准定义。

- 常量`math.MaxFloat32`表示`float32`能表示的最大数值，大约是 `3.4e38`；对应的`math.MaxFloat64`常量大约是`1.8e308`。
- 一个`float32`类型的浮点数可以提供大约6个十进制数的精度，而`float64`则可以提供约15个十进制数的精度



小数点前面或后面的数字都可能被省略（例如`.707`或`1.`）

~~~go
const e = 2.71828 // (approximately)
const Avogadro = 6.02214129e23  // 阿伏伽德罗常数
const Planck   = 6.62606957E-34 // 普朗克常数
~~~



我们看一下float类型的误差问题：

~~~go
var f float32 = 16777216 // 1 << 24
fmt.Println(f == f+1)    // "true"!
~~~

要处理这种误差问题，第一种方案是使用数值算法，第二种是使用大整数进行模拟。



### 复数

Go语言提供了两种精度的复数类型：`complex64`和`complex128`，两者分别由`float32`和`float64`来实现的。内置的`complex`函数用于构建复数，内建的`real`和`imag`函数分别返回复数的实部和虚部：

~~~go
var x complex128 = complex(1, 2) // 1+2i
var y complex128 = complex(3, 4) // 3+4i
fmt.Println(x*y)                 // "(-5+10i)"
fmt.Println(real(x*y))           // "-5"
fmt.Println(imag(x*y))           // "10"
~~~

如果一个浮点数面值或一个十进制整数面值后面跟着一个i，例如`3.141592i`或`2i`，它将构成一个复数的虚部。

Go语言中的复数还支持`+`、`-`、`*`、`\`。我们可以通过`+`来将一个复数虚部字面量可以加到另一个普通数值字面量上，从而构成一个复数。

~~~go
x := 1 + 2i
y := 4i + 3 // OK
~~~

复数也可以用==和!=进行相等比较。只有两个复数的实部和虚部都相等的时候它们才是相等的（注意，实部和虚部是由浮点数来实现的）。

math/cmplx包提供了复数处理的许多函数，例如求复数的平方根函数和求幂函数。

```go
fmt.Println(cmplx.Sqrt(-1)) // "(0+1i)"
```

### 布尔

一个布尔类型的值只有两种：true和false。

在Go语言中，不支持隐式转换。因此布尔值并不会隐式转换为数字值0或1，反之亦然。

### 字符串

一个字符串是一个**不可改变的**字节序列。内置的`len`函数可以返回一个字符串的字节个数（不是`Unicode`字符个数）。索引操作`s[i]`返回第`i`个字节的字节值

~~~go
s := "hello, world"
fmt.Println(len(s))     // "12"
fmt.Println(s[0], s[7]) // "104 119" ('h' and 'w')
~~~

如果试图访问超出字符串索引范围的字节将会导致panic异常：

~~~go
c := s[len(s)] // panic: index out of range
~~~

因为字符串是不可修改的，因此尝试修改字符串内部数据的操作也是被禁止的：

```go
s[0] = 'L' // compile error: cannot assign to s[0]
```



通过`s[i:j]`来返回子串，区间是左闭右开的。`i`、`j`都可以被忽略。其中`i`的默认值是`0`，而`j`的默认值是`len(s)`。

~~~go
fmt.Println(s[0:5]) // "hello"
fmt.Println(s[:5]) // "hello"
fmt.Println(s[7:]) // "world"
fmt.Println(s[:])  // "hello, world"
~~~

由于string类型的不可修改性，所以两个字符串共享相同数据的行为也是安全的。而且字符串切片操作代价也是低廉的。

![img](assets/ch3-04.png)



其中+操作符将两个字符串连接构造一个新字符串：

```go
fmt.Println("goodbye" + s[5:]) // "goodbye, world"
```

字符串可以用`==`和`<`进行比较，其中会逐个字节进行比较。



因为Go语言源文件总是用UTF8编码，所以Go语言的字符串字面量以UTF-8来编码。如果想要在字符串面值中包含二进制字节数据，那么使用转义字符`\xhh`，或者`\ooo`。其中两个`h`表示十六进制数字（大写或小写都可以），o三个`o`表示八进制数字，但不能超过`\377`，即十进制`255`。



Go语言中的转义字符如下：

~~~
\a      响铃
\b      退格
\f      换页
\n      换行
\r      回车
\t      制表符
\v      垂直制表符
\'      单引号（只用在 '\'' 形式的rune符号面值中）
\"      双引号（只用在 "..." 形式的字符串面值中）
\\      反斜杠
~~~



原生的字符串字面量的语法为

~~~go
const GoUsage = `Go is a tool for managing Go source code.

Usage:
	go command [arguments]
...`
~~~

在原生字面量内，转义序列不起作用。原生字符串包括所有的文本内容，甚至包括反斜杠和换行。唯一的特殊处理是，回车符会被删除（换行符会被保留）。

原生字符串字面量广泛应用于HTML模板、JSON面值、命令行提示信息等需要多行或者有大量转义字符的场景。



Unicode（ [http://unicode.org](http://unicode.org/) ）字符集包括了世界上大部分文字符号，甚至包括重音符号。每个符号都分配一个唯一的Unicode码点。

在Go语言中，提供了`rune`类型（与`int32`类型等价）来表示一个Unicode码点。显然，rune类型可以支持UTF32编码方案，但是对于UTF8编码方案来说过于浪费。

UTF8是变长编码方案，用1到4个字节来表示每个Unicode字符。它有一个前缀编码来支持变长方案

~~~
0xxxxxxx                             runes 0-127    
110xxxxx 10xxxxxx                    128-2047       
1110xxxx 10xxxxxx 10xxxxxx           2048-65535     
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx  65536-0x10ffff 
~~~

在Go语言中，可以使用`\xhh`（对应8bit编码值）、`\uhhhh`（对应16bit编码值）或者`\Uhhhhhhhh`（对应32bit的编码值）来表示一个Unicode字符。

~~~go
"世界"
"\xe4\xb8\x96\xe7\x95\x8c"
"\u4e16\u754c"
"\U00004e16\U0000754c"
~~~

上面这四种表达方式都是一样的。



len内置函数只能获取字符串的字节长度，要获取字符长度要使用`utf8.RuneCountInString`

~~~go
import "unicode/utf8"

s := "Hello, 世界"
fmt.Println(len(s))                    // "13"
fmt.Println(utf8.RuneCountInString(s)) // "9"
~~~



`utf8.DecodeRuneInString`函数接收一个字符串作为输入参数，返回首个有效Unicode字符对应的rune值（即其Unicode编码）以及该字符的字节长度。

```go
for i := 0; i < len(s); {
	r, size := utf8.DecodeRuneInString(s[i:])
	fmt.Printf("%d\t%c\n", i, r)
	i += size
}
```

幸运的是，Go语言的range循环在处理字符串的时候，会自动隐式地解码UTF8字符串。

![img](assets/ch3-05.png)





string与[]rune的类型转换

~~~go
~~~



### 常量



### 运算符

下面是Go语言中关于算术运算、逻辑运算和比较运算的二元运算符，它们按照优先级递减的顺序排列：

```go
*      /      %      <<       >>     &       &^
+      -      |      ^
==     !=     <      <=       >      >=
&&
||
```

在同一个优先级，使用左优先结合规则。但是使用括号`()`可以提升优先级。

运算的一些行为

- 在Go语言中，%取模结果的符号和被取模数的符号总是一致的。
- 只有整型支持取模操作%
- 整数除法会向着0方向截断余数。



关于比较运算符的说明

- "==" 运算符可以用于比较所有的基本数据类型，如 int, float, string 等。它也可以在比较 struct 类型的变量时使用，只要它们的每个成员都是可比较的。
- 然而，“==”不能直接用于比较数组、切片、映射或函数。
- "==" 要求两个操作数的类型相同。



Go语言还提供了以下的bit位操作运算符，前面4个操作运算符并不区分是有符号还是无符号数：

```go
&      位运算 AND
|      位运算 OR
^      位运算 XOR
&^     位清空（AND NOT）
<<     左移
>>     右移
```

在`x<<n`和`x>>n`移位运算中，决定了移位操作的bit数部分必须是无符号数；被操作的x可以是有符号数或无符号数。

- 左移运算用0填充右边空缺的bit位，

- 无符号数的右移运算也是用0填充左边空缺的bit位，

- 有符号数的右移运算会用符号位的值填充左边空缺的bit位。





## 复合类型



## 结构

~~~go
if condition {
    // code to execute if condition is true
} else {
    // code to execute if condition is false
}
~~~



四种for语句

~~~go
for initialization; condition; post {
    // code to be executed
}

for condition {
    // code to be executed
}

for {
    // code to be executed
}

for key, value := range map {
    // code to be executed
}
~~~



## 函数

一个函数的声明由

- 函数名字
- 形参列表
- 可选的返回值列表
- 函数体（函数定义）

