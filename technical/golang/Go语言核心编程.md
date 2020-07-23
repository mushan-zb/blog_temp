# Go 语言核心编程 作者：李文塔 - 读书笔记

## <a id="chapter3">第三章 - 类型系统</a>

### <a id="type-description">类型简介</a>
- 命名类型：通过标识符表示 例：布尔型 bool、整型 int int64、浮点型 float64、复数 complex128、字符 byte rune、字符串 string、接口 error、自定义类型 type new_type old_type
- 未命名类型：由预声明类型、关键字和操作符组成 例：数组 array、切片 slice、字典 map、指针 pointer、通道 channel、结构 struct、接口 interface、函数 function

**底层类型（underlying type）的规则**
1. 预声明类型（Pre-declared types）和类型字面量（type literals）的底层类型是它们自身
2. 自定义类型（type newtype oldtype）中 newtype 的底层类型是逐层递归向下查找的，直到查找到 oldtype 是预声明类型或者类型字面量为止

> **注：** 未命名类型、类型字面量和 Go 语言基本类型中的符合类型等价

``` demo-3.1
type T1 string
type T2 T1
type T3 []string
type T4 T3
type T5 []T1
type T6 T5
```
变量    | T1     | T2     | T3       | T4       | T5   | T6 
--------|--------|--------|----------|----------|------|------
底层类型 | string | string | []string | []string | []T1 | []T1

> 理解：[]T1 为未命名类型，即为类型字面量，所以符合规则一

**类型相同的规则**
1. 两个命名类型相同的条件是两个类型声明的语句完全相同
2. 命名类型和未命名类型永远不相同
3. 两个未命名类型相同的条件是它们的类型声明字面量的结构相同，并且内部元素的类型相同
4. 通过类型别名语句声明的两个类型相同  
    Go 1.9 引入了类型别名语法 type T1 = T2，T1 的类型完全和 T2 一样，引入别名的原因：
    1. 为了解决新旧包的迁移兼容问题
    2. Go 的按包进行隔离的机制不太精细，有时我们需要将大包划分成几个小包进行开发，但需要在大包里暴露全部类型给使用者
    3. 解决新旧类型的迁移问题，新类型先是旧类型的别名，后续软件基于新类型编程，在合适时间将新类型升级为和旧类型不兼容

**类型赋值的条件（T1 赋值给 T2）**
1. T1 和 T2 的类型相同
2. T1 和 T2 具有相同的底层类型，并且 T1 和 T2 里面至少有一个是未命名类型
3. T2 是接口类型，T1 是具体类型，它们拥有相同的元素类型，并且 T1 和 T2 中至少有一个是未命名类型
4. T1 和 T2 都是管道类型，它们拥有相同的元素类型，并且 T1 和 T2 中至少有一个是未命名类型
5. T1 是预声明标识符 nil，T2 是 pointer、function、slice、map、channel、interface 类型
6. T1 是一个字面常量值，可以用来表示类型 T 的值
``` demo-3.2
package main

import "fmt"

type Map map[string]string
func (m Map)Print() {
    for key, value := range m {
        fmt.Println(key, value)
    }
}

type iMap Map
// 只要底层类型是 slice、map 等支持 range 的类型字面量，新类型仍然可以使用 range 迭代
func (m iMap)Print() {
    for key, value := range m {
        fmt.Println(key, value)
    }
}

type slice []int
func (s slice)Print() {
    for _, value := range s {
        fmt.Println(value)
    }
}

func main() {
    mp := make(map[string]string, 10)
    mp["key"] = "value"

    // mp 和 ma 有相同的底层类型 map[string]string，并且 mp 是未命名类型
    // 所以 mp 可以直接赋值给 ma
    var ma Map = mp

    // im 和 ma 虽然有相同的底层类型 map[string]string，但没有一个是未命名类型，不能赋值
    // var im iMap = ma // 编译不通过

    ma.Print()
    im.Print()

    // Map 实现了 Print()，所以可以赋值给接口类型变量
    var i interface {
        Print()
    } = ma

    i.Print()

    s1 := []int{1, 2, 3}
    var s2 slice = s1
    s2.Print()
}
```

**强制类型转换的条件（x 强制转化给类型 T）**
1. x 可以直接赋值给 T 类型变量
2. x 的类型和 T 具有相同的底层类型
``` deome-3.3
// 借用上面 Map、iMap 的声明代码
func main() {
    mp := make(map[string]string, 10)
    mp["key"] = "value"
    var ma Map = mp

    // var im iMap = ma // 不能直接赋值
    // 强制类型转换
    var im iMap = iMap(ma)

    ma.Print()
    im.Print()
}
```
3. x 的类型和 T 都是未命名的指针类型，并且指针指向的类型具有相同底层类型
4. x 的类型和 T 都是整型、浮点型、复数类型
5. x 是整数值或 []byte 类型的值，T 是 string 类型
6. x 是一个字符串，T 是 []byte 或 []rune
```demo-3.4
s ;= "你好, mushan"
var a []byte = []byte(s)
var b string = string(a)
var c []rune = []rune(s)
fmt.Printf("%T\n", a) // []uint8 byte 是 uint8 的别名
fmt.Printf("%T\n", b) // string
fmt.Printf("%T\n", c) // []int32 rune 是 int32 的别名
```
> **注：**  数值类型和 string 类型之间转换可能造成值部分丢失，string 和数值之间转换建议使用标准库 strconv。Go 不支持指针和 integer 直接转换，可以使用标准库 unsafe 包进行处理

### <a id="type-method">类型方法</a>
**自定义类型**

使用 type 关键字可以进行自定义类型，自定义的类型是命名类型。格式：type newtype oldtype  
> **注：** 使用 type 定义的新类型不会继承旧类型的方法

常用的自定义类型为自定义 struct 类型
``` demo-3.5
// 使用 type 自定义的结构类型格式
// type name struct {
//     Field1 type1
//     Field2 type2
//     ......
// }

// struct 初始化
type Person struct {
    name string
    age int
}

// 不推荐写法 - 按照字段顺序进行初始化
a := Person{"mushan", 18}

// 推荐写法 - 指定字段名进行初始化
x := Person{name:"mushan", age:18}
y := Person{
    name: "mushan",
    age: 18,
}
z := Person{
    name: "mushan",
    age: 18}

// 推荐 - 构造函数初始化 - 标准库中 errors 的 New 函数示例
func New(text string) error{
    return &errorString{text}
}
type errorString struct{
    s string
}
```
> **注：** 如果初始化的结尾 "}" 独占一行，则最后一个字段的后面必须有逗号

**结构字段的特点**  
结构字段可以是任意类型，基本类型、接口类型、指针类型、函数类型都可以作为 struct 的字段。结构字段的类型名必须唯一，struct 字段类型可以是普通类型也可以是指针类型

**匿名类型**  
在定义 struct 的过程中，如果字段只给出字段类型，没有给出字段名，则称为匿名字段。被匿名嵌入的字段必须是命名类型或者命名类型的指针，类型字面量不能作为匿名字段使用。匿名字段的字段名默认就是类型名，如果匿名字段是指针类型，则默认的字段名就是指针指向的类型名。但一个结构体里不能同时存在某一类型及其指针类型的匿名字段，原因是二者的字段名相同。如果嵌入的字段来自其他包，则需要加上包名

**自定义接口类型**  
接口字面量是非命名类型，自定义接口类型是命名类型。如果一个 struct 的方法实现了接口类型里的所有方法，则称该 struct 实现了这个接口
``` demo-3.6
// interface{} 是接口字面量类型标识，所以 i 是非命名类型变量
var i interface{}

// Reader 是自定义接口类型，属于命名类型
type Reader interface{
    Read(p []byte) (n int, err error)
}
```

**方法**  
前面介绍的 Go 语言的类型系统和自定义类型，仅使用类型对数据进行抽象和封装。本节介绍 Go 语言的类型方法，是对类型行为的封装。Go 语言的方法非常简单，可以看作特殊类型的函数，其显示地将对象实例或指针作为函数的第一个参数，并且参数名可以自定义，不强制要求一定是 this 或者 self。这个对象实例或者指针称为方法的接收者（receiver）
``` demo-3.7
// 类型方法的接收者是值类型
func (t TypeName)MethodName(ParamList) (ReturnList) {
    // method body
}

// 类型方法接收者是指针
func (t *TypeName)MethodName(ParamList) (ReturnList) {
    // method body
}
```
说明：
- t 是接收者，可以自定义
- TypeName 为命名类型的类型名
- MethodName 为方法名，是一个自定义标识符
- ParamList 是形参列表
- ReturnList 是返回值列表

Go 语言的类型方法本质上就是一个函数，没有使用隐式指针。可以将类型方法改写成常规函数，如下：
``` demo-3.8
// 类型方法的接收者是值类型
func MethodName(t TypeName， OtherParamList) (ReturnList) {
    // method body
}

// 类型方法接收者是指针
func MethodName(t *TypeName， OtherParamList) (ReturnList) {
    // method body
}

// 示例
type SliceInt []int

func (s SliceInt)Sum() int {
    sum := 0
    for _, i := range s {
        sum += i
    }
    return sum
}

// 等价方法
func Sum(s SliceInt) int {
    sum := 0
    for _, i := range s {
        sum += i
    }
    return sum
}
```
**类型方法的特点**
1. 可以为命名类型增加方法（接口除外），非命名类型不能自定义方法  
**注：** 接口类型本身是一个方法的签名集合，不能增加具体的实现方法
2. 为类型增加方法有一个限制，方法必须和类型定义在同一个包中  
**注：** 不能为 int bool 等预声明类型增加方法，因为它们是 Go 语言内置的预声明类型，作用域是全局
3. 方法的命名空间可见性和变量一致，大写开头的方法可以在包外访问，否则只能在包内可见
4. 使用 type 定义的自定义类型是新类型，新类型不继承原有类型的方法，但是底层类型支持的运算可以被继承

``` demo-3.9
type Map map[string]string

func (m Map)Print() {
    // 底层类型支持 range 运算，新类型可用
    for key, value := range m {
        fmt.Println(key, value)
    }
}

type MyInt int

func main() {
    var a MyInt = 10
    var b MyInt = 10
    // int 类型支持的加减乘除运算，新类型同样可用
    c := a + b
    d := a * b
    fmt.Printf("%d, %d", c, c)
}
```

### <a id="method-calls">方法调用</a>
本节内容：类型方法的调用方式、方法集、方法变量和方法表达式

**一般调用方式**  
`TypeInstanceName.MethodName(ParamList)`
- TypeInstanceName: 类型实例名或者指向实例的指针变量名
- MethodName: 类型方法名
- ParamList: 方法实参

**方法值**  
变量 x 的静态类型是 T，M 是类型 T 的一个方法， x.T 被称为方法值。x.T 是一个函数类型变量，可以赋值给其他变量，并像普通函数名一样使用
``` demo-3.10
f := x.M
f(arg...)
// 等价于
x.M(arg...)
```
方法值其实是一个带有闭包的函数变量，其底层实现原理和带有闭包的匿名函数类似，接收值被隐式绑定到方法值的闭包环境中。后续调用不需要显示传递接收者

**方法表达式**  
方法表达式相当于提供一种语法将类型方法调用显式地转换为函数调用，接收者（receiver）必须显式地传递进去
``` demo-3.11
type T struct {
    a int
}

func (t *T)Set(i int) {
    t.a = i
}

func (t T)Get() int {
    return t.a
}

func main() {
    t := T{a: 1}

    // 方法表达式调用
    *T.Set(&t, 1)
    f2 := T.Set
    f2(&t, 1)

    // 普通方法调用
    t.Get()
    // 方法表达式调用
    T.Get(t)
    f1 := T.Get
    f1(t)
}
```
表达式 T.Get 和 *T.Set 被称为方法表达式，方法表达式可以看作函数名，只不过函数的首个参数是接收者的实例或指针。T.Get 的函数签名是 func(t T) int，*T.Set 的函数签名是 func(t *T, i int)

**方法集**  
命名类型方法接收者有两种类型，一个是值类型，另一个是指针类型。但方法和函数的实参传递都是值拷贝。如果接收者是值类型，则传递的是值的副本；如果是指针类型，则传递的是指针的副本（使用值或指针都可以调用 - 编译器会根据类型自动转换）  
将接收者（receiver）为值类型 T 的方法集合记录为 S，将接收者（receiver）为指针类型 *T 的方法集合记录为 *S
- T 类型的方法集是 S
- *T 类型的方法集是 S 和 *S

**值调用和表达式调用方法集**
- 通过类型字面量显示地进行值调用和表达式调用，编译器不会做自动转换，会进行严格的方法集检查
``` demo-3.12
type Data struct{}

func (Data)TestValue() {}
func (*Data)TestPointer() {}

// 字面量显式调用，无论值调用，还是表达式调用
// 编译器都不会进行方法集的自动转换，编译器会严格校验方法集
// Data 的方法集是 TestValue
// *Data 的方法集是 TestValue 和 TestPointer

(*Data)(&struct{}{}).TestPointer() // 显式调用
(*Data)(&struct{}{}).TestValue() // 显式调用

(Data)(struct{}{}).TestValue() // 方法值
Data.TestValue(struct{}{}) // 方法表达式

// 以下调用因为方法集不匹配而失败
// Data.TestPointer(struct{}{}) // Data 类型没有 TestPointer 方法
// Data(struct{}{}).TestPointer() // Data 字面量不能调用指针方法
```
- 通过类型变量进行值调用和表达式调用，值调用编译器会进行自动转换；表达式调用编译器不会自动转换，会进行严格的方法集检查
``` demo-1.13
// 声明类型变量
var data Data = struct{}{}

// 表达式调用编译器不会进行自动转换
Data.TestValue(data)
// Data.TestValue(&data) // 不能将 *Data 类型传递给 Data
(*Data).TestPointer(&data)
// Data.TestPointer(&data) // Data 类型没有 TestPointer 方法

// 值调用会进行自动转换
data.TestValue()
(&data).TestValue()
data.TestPointer()
(&data).TestPointer()
```

### <a id="combination-and-method-set">组合和方法集</a>
结构类型（struct）为 Go 提供了强大的类型扩展，第一：struct 可以嵌入任意其他类型的字段；第二：struct 可以嵌套自身的指针类型

**组合**  
type 定义的新类型不会继承原有类型的方法。但可以使用命名结构类型嵌套其他命名类型的字段，外层的结构类型是可以调用嵌入字段类型的方法。既可以是显示调用，也可以是隐式调用  
struct 中的组合非常灵活，可以表现为水平的字段扩展，也可以表现为分层次扩展。struct 类型中的字段称为“内嵌字段”  
内嵌字段的初始化和访问
``` demo-3.14
type X struct {
    a int
}

type Y struct {
    X
    b int
}

type Z struct {
    Y
    c int
}

func main() {
    x := X{a: 1}
    y := Y{
        X: x,
        b: 2,
    }
    z := Z{
        Y: y,
        c: 3,
    }

    // 只要内嵌字段唯一，不需要使用全路径进行访问
    // z.a z.Y.a z.Y.X.a 三者等价
    fmt.Println(z.a, z.Y.a, z.Y.X.a) // 输出：1 1 1
    
    z = Z{}
    z.a = 2
    fmt.Println(z.a, z.Y.a, z.Y.X.a) // 输出：2 2 2
}
```
**注：** 在 struct 的多层嵌套中，不同嵌套层次可以有相同字段，此时最好使用完全路径进行访问和初始化。在实际数据结构的定义中应尽量避开相同字段，避免使用中出现歧义

内嵌字段的方法调用  
struct 类型方法调用也是用点操作，不同嵌套层次的字段可以有相同的方法，外层变量调用内嵌字段的方法也可以使用简化模式。但如果外层字段和内层字段有相同的方法，则使用简化模式访问外层方法会覆盖内层方法。类似面向对象中子类覆盖父类的同名方法
``` demo-3.15
type X struct {
    a int
}
func (x X)Print() {
    fmt.Printf("In X, a=%d\n", x.a)
}
func (x X)XPrint() {
    fmt.Printf("In X, a=%d\n", x.a)
}

type Y struct {
    X
    b int
}
func (y Y)Print() {
    fmt.Printf("In Y, b=%d\n", y.b)    
}

type Z struct {
    Y
    c int
}
func (z Z)Print() {
    fmt.Printf("In Z, c=%d\n", z.c)
}

func main() {
    x := X{a: 1}
    y := Y{
        X: x,
        b: 2,
    }
    z := Z{
        Y: y,
        c: 3,
    }

    // 从外向里查找，首先找到的是 Z 的 Print() 方法
    z.Print()
    // 从外向里查找，最终找到的是 X 的 Print() 方法
    z.XPrint()
    z.Y.XPrint()
}
```

**组合的方法集**
1. 若类型 S 包含匿名字段 T，则 S 的方法集包含 T 的方法集
2. 若类型 S 包含匿名字段 *T，则 S 的方法集包含 T 和 *T 的方法集
3. 不管类型 S 中嵌入的匿名字段是 T 还是 *T，*S 的方法集总是包含 T 和 *T 的方法集
``` demo-3.16
type X struct {
    a int
}
func (x X)Get() int {
    return x.a
}

func (x *X)Set(a int) {
    x.a = a
}

type Y struct {
    X
}
type Z struct {
    *X
}

func main() {
    x := X{a: 1}
    y := Y{X: x}
    fmt.Println(y.Get()) // 1

    // 编译器做了自动转换
    y.Set(2)
    fmt.Println(y.Get()) // 2

    // 方法表达式调用，编译器不会自动转换
    // Y 内嵌 X，所以 type Y 的方法集是 Get，type *Y 的方法集是 Set Get
    (*Y).Set(&y, 3)

    // type Y 的方法集合并的方法没有 Set 方法，所以编译不通过
    // Y.Set(y, 3)
    fmt.Println(y.Get()) // 3
    
    z := Z{X: &x}
    // Z 内嵌 *X，所以 type Z 和 type *Z 方法集都包含类型 X 定义的方法 Get 和 Set
    // 为不让编译器自动转换，使用方法表达式调用方式
    Z.Set(z, 4)
    fmt.Println(z.Get()) // 4

    (*Z).Set(&z, 5)
    fmt.Println(z.Get()) // 5
}
```
**注：** Go 编译器自动转换仅适用于直接通过类型实例调用才有效，类型实例传递给接口时，编译器不会进行自动转换，而是进行严格的方法集校验

### <a id="function-type">函数类型</a>
函数类型分两种：函数字面量类型（未命名类型），函数命名类型

**函数字面量类型**  
函数字面量类型的语法表达格式：func (InputTypeList)(OutputTypeList)，有名函数和匿名函数的类型都属于函数字面量类型。有名函数的定义相当于初始化一个函数字面量赋值给一个函数名变量。从 Go 类型系统的角度看，有名函数和匿名函数都是函数字面量类型的实例

**函数命名类型**  
`type NewFuncType FuncLiteral` NewFuncType 为新定义的函数命名类型，FuncLiteral 为函数字面量，是 NewFuncType 的底层类型

**函数签名**  
函数签名就是有名函数或匿名函数的字面量类型。所以有名函数和匿名函数的函数签名可以相同，函数签名是函数的字面量类型，不包括函数名

**函数声明**  
Go 语言没有函数声明，可以直接调用，但 Go 调用汇编语言编写的函数还是要使用函数声明语句
``` demo-3.17
// 函数声明 = 函数名 + 函数签名
// 函数签名
func (InputTypeList)(OutputTypeList)
// 函数声明
func FuncName(InputTypeList)(OutputTypeList)
```
字面量类型是一种未命名类型（unnamed type），不能定义自己的方法，所以必须显示地使用 type 声明一个有名函数类型，然后为其添加方法
``` demo-3.18
// src/net/http/server.go http 标准库对函数类型的实现

// 定义一个有名函数类型 HandlerFunc
type HandlerFunc func(ResponseWriter, *Request)

// 为有名函数类型添加方法 这是一中包装器的编程技法
// ServeHTTP calls f(w, r)
func (f HandlerFunc)ServeHTTP(w ResponseWriter, r *Request) {
    // 拦截或过滤操作之后回调函数
    f(w, r)
}

// 函数类型 HandlerFunc 实现了接口 Handler 的方法
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

func (mux *ServeMux)Handle(pattern string, handler Handler)

// HandlerFunc 类型的变量可以传递给 Handler 接口变量
func (mux *ServeMux)HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    mux.Handle(pattern, HandlerFunc(handler))
}
```
函数类型的意义：
- 函数也是一种类型，可以再函数字面量类型的基础上定义一种命名函数类型
- 有名函数和匿名函数的函数签名与命名函数类型的底层类型相同，可以进行类型转换
- 可以为有名函数类型添加方法，可以方便的为一个函数增加“拦截”或者“过滤”等额外功能，提供了一中装饰设计模式
- 为有名函数增加方法，使其与接口打通关系，使用接口的地方可以传递函数类型的变量

## <a id="chapter4">接口</a>
### <a id="basic-concepts">基本概念</a>
**接口声明**
``` demo-4.1
// 接口字面量类型
interface {
    MethodSignature1
    MethodSignature2
}
// 接口命名类型
type InterfaceName interface {
    MethodSignature1
    MethodSignature2
}
```
接口定义大括号内可以是方法声明的集合，也可以嵌入另一个接口类型匿名字段
```demo-4.2
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}

// 如下三种声明等价，且最终会被 Go 编译器自动展开处理，成为第三种格式
type ReadWriter interface {
    Reader
    Writer
}
type ReadWriter interface {
    Reader
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```
声明新接口类型的特点
- 接口命名一般以“er”结尾
- 接口定义的内部方法声明不需要 func 引导
- 在接口定义中，只有方法声明没有方法实现

**接口初始化**  
单纯地声明一个接口变量没有任何意义，接口只有被初始化为具体的类型时才有意义。接口作为一个胶水层或者抽象层，起到抽象和适配的作用。没有初始化的接口变量，其默认值为 nil  
接口变量的两种初始化方法：
- 实例赋值接口  
    如果具体类型实例的方法集是某个接口的方法集的超集，则称该具体类型实现了接口，可以将该具体类型的实例直接赋值给接口类型的变量，编译器会进行静态的类型检查。接口被初始化后，调用接口的方法等价于调用接口绑定的具体类型的方法
- 接口变量赋值接口变量
    已经初始化的接口类型变量 a 直接赋值给另一个接口变量 b，要求 b 的方法集是 a 的方法集的子集，编译器同样会进行静态的类型检查。此时接口变量 b 绑定的具体实例是接口变量 a 绑定的具体实例的副本

**接口方法调用**  
接口调用的最终地址是在运行期决定的，将具体类型变量赋值给接口后，会使用具体类型的方法指针初始化接口变量，当调用接口变量的方法时，实际上是间接地调用实例方法。接口方法调用不是一种直接的调用，有一定的运行时开销  
直接调用未初始化的接口变量的方法会产生 panic

**接口的动态类型和静态类型**
- 接口绑定的具体实例类型称为接口的动态类型。接口可以绑定不同类型的实例，所以接口的动态类型会随着其绑定的不同类型实例而发生变化
- 接口在被定义时，其类型就已经被确定，这个类型叫做接口的静态类型。其本质特征就是接口的方法签名集合

### <a id="interface-operation">接口运算</a>
**类型断言**  
接口类型断言的语法：`i.(TypeName)` i 必须是接口变量，如果是具体类型变量，则编译器会报 non-interface type xxx on left，TypeName 可以是接口类型名，也可以是具体类型名
- TypeName 是具体类型名，则类型断言用于判断接口变量 i 绑定的实例类型是否就是具体类型
- TypeName 是接口类型名，则类型断言用于判断接口变量 i 绑定的实例类型是否也实现了 TypeName 接口

接口类型的适用语法：
``` demo-4.3
o := i.(TypeName) // 如果 TypeName 类型不满足条件，程序抛出 panic
o, ok := i.(TypeName) // 如果 TypeName 类型不满足条件，ok 为 false
```

**类型查询**
```
switch v := i.(type) {
    case type1:
        xxx
    case type2:
        xxx
    case type3:
        xxx
    default:
        xxx
}
```
- 类型查询和类型断言具有相同的语义，只是语法格式不同。两者都能判断接口变量绑定的实例的具体类型，以及判断接口变量绑定的实例是否满足另一个接口类型
- 类型查询使用 case 子句一次判断多个类型，类型断言一次只能判断一个类型，也可以使用 if else if 语句达到类型同样效果

**接口优点和使用形式**  
- 接口优点
  - 解耦：复杂系统进行垂直和水平的分割是常用的设计手段，在层与层之间使用接口进行抽象和解耦是一种好的编程策略
  - 实现泛型：使用空接口作为函数或方法参数能够用在需要泛型的场景中
- 使用形式
  - 作为结构内嵌字段
  - 作为函数或方法的形参
  - 作为函数或方法的返回值
  - 作为其他接口定义的嵌入字段

### <a id="empty-interface>空接口</a>
**基本概念**
空接口：没有任何方法的接口，表示为 interface{}。 系统中任何类型都符合空接口的要求，包括基本类型 int、float、string 等

**空接口的用途**  
空接口和泛型  
Go 语言没有泛型，如果一个函数需要接收任意类型的参数，则参数类型可以使用空接口类型
```
// 典型例子，fmt 标准包中的 print 函数
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
```
空接口和反射  
空接口是反射实现的基础，反射库就是将相关具体的类型转换并赋值给空接口后才去处理

### <a id="interface-implementation">接口内部实现</a>
**注：** 空接口不是真的为空，接口有类型和值两个概念

此处内容暂时略过，书本 P120 - P132

## <a id="chapter5">并发</a>
### <a id="concurrent-basic">并发基础</a>
- 并行：程序在任意时刻都是同时运行的
    并行是在任一粒度的时间内都具备同时执行的能力：最简单的并行就是多机，多台机器并行处理
- 并发：程序在单位时间内是同时运行的
    并发是在规定时间内多个请求都得到处理，实际上可能是分时操作。并发重在避免阻塞，使程序不会因为一个阻塞而停止处理。典型的并发应用：分时操作系统（忽略多核 CPU）

并行具有瞬时性，并发具有过程性；并发在于结构，并行在于执行。应用程序具备好的并发结构，操作系统才能更好地利用硬件并行执行，同时避免阻塞等待，合理地进行调度，提高 CPU 利用率

**goroutine**  
goroutine 被翻译成 go 例程或 go 协程  
操作系统可以进行线程和进程的调度，本身具备并发处理能力，但切换代价过于高昂，进程切换需要保存现场，耗费较多的时间。Go 语言的并发是基于用户层在构筑一级调度  
**注：** go 关键字后面必须跟一个函数，不能是语句或者其他东西，函数的返回值被忽略

goroutine 的特性：
- go 的执行是非阻塞的，不会等待
- go 后面的函数的返回值会被忽略
- 调度器不保证多个 goroutine 的执行次序
- 没有父子 goroutine 的概念，所有的 goroutine 都是平等地被调度和执行
- Go 程序在执行时会单独为 main 函数创建一个 goroutine，遇到其他 go 关键字时再去创建其他的 goroutine
- Go 没有暴露 goroutine id 给用户，所以不能在一个 goroutine 里面显式地操作另一个 goroutine，不过 runtime 包提供了一些函数访问和限制 goroutine 的相关信息

**channel**  
channel 被翻译成通道  
goroutine 是 Go 语言里面的并发执行体，通道是 goroutine 之间通信和同步的重要组件。Go 的哲学：不要通过共享内存来通信，而是通过通信来共享内存。通道是 Go 通过通信来共享内存的载体

通道是有类型的，可以理解成有类型的管道。声明一个通道变量没有意义，并没有被初始化，其值是 nil。Go 语言使用 make 函数创建通道
``` demo-5.1
// 创建无缓冲的通道，通道存放元素类型为 datatype
make(chan datatype)

// 创建有 10 个缓存的通道
make(chan datatype, 10)
```
通道分为无缓冲通道和有缓冲通道，Go 提供内置函数 len 和 cap，无缓冲通道的 len 和 cap 都是 0，有缓冲通道的 len 代表没有被读取的元素数，cap 代表整个通道的容量。无缓冲通道既可以用于通信也可以用于两个 goroutine 同步，有缓冲的通道主要用于通信

操作不同状态的 chan 会引发三种行为
- panic
  - 向已经关闭的通道写数据会导致 panic。最好由写入者关闭通道，能最大程度地避免向已经关闭的通道写数据而导致 panic
  - 重复关闭通道会导致 panic
- 阻塞
  - 向未初始化的通道写数据或读数据都会导致当前 goroutine 的永久阻塞
  - 向缓冲区已满的通道写入数据会导致 goroutine 阻塞
  - 通道中没有数据，读取该通道会导致 goroutine 阻塞
- 非阻塞
  - 读取已经关闭的通道不会引发阻塞，而是立即返回通道元素类型的零值，可以使用 `data, ok := channel` 语法判断通道是否已经关闭
  - 向有缓冲且没有满的通道读写都不会引发阻塞

**WaitGourp**  
WaitGourp 用来等待多个 goroutine 完成，main goroutine 调用 Add 设置需要等待 goroutine 的数目，每一个 goroutine 结束时调用 Done(),Wait() 被 main 用来等待所有的 goroutine 完成
``` demo-5.2
type WaitGourp struct {
    // contains filtered or unexported fields
}

// 添加等待信号
func (wg *WaitGourp)Add(delta int)

// 释放信号
func (wg *WaitGourp)Done()

// 等待
func (Wg *WaitGourp)Wait()
```

**select**  
select 是类 UNIX 系统提供的一个多路复用系统 API，Go 语言借用多路复用的概念，提供了 select 关键字用于多路监听多个通道。当监听的通道没有状态是可读或可写，select 是阻塞的；只要监听的通道中有一个状态是可读或可写的，则 select 就不会阻塞，而是进入处理就绪通道的分支流程。如果监听的通道有多个可读或可写的状态，则 select 随机选取一个处理
- 扇入，指将多路通道聚合到一条通道中处理，最简单的是使用 select 聚合多条通道服务
- 扇出，指将一条通道发散到多条通道中处理，Go 语言中使用 go 关键字启动多个 goroutine

**通知退出机制**
读取已经关闭的通道不会引起阻塞，也不会导致 panic，而是立即返回该通道存储类型的零值。关闭 select 监听的某个通道能使 select 立即感知这种通知，然后进行处理。这就是通知退出机制

### 并发范式
**生成器**  
- 最简单的带缓存的生成器
``` demo-5.3
package main

import (
    "fmt"
    "math/rand"
)

func GenerateInt() chan int {
    ch := make(chan int, 10)
    // 启动一个 goroutine 用于生成随机数，函数返回一个通道用于获取随机数
    go func() {
        for {
            ch <-rand.Int()
        }
    }()
    return ch
}

func main() {
    ch := GenerateInt()
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```
- 多个 goroutine 增强型生成器
``` demo-5.4
package main

import (
    "fmt"
    "math/rand"
)

func GenerateIntA() chan int {
    ch := make(chan int, 10)
    go func() {
        for {
            ch <-rand.Int()
        }
    }()
    return ch
}

func GenerateIntB() chan int {
    ch := make(chan int, 10)
    go func() {
        for {
            ch <-rand.Int()
        }
    }()
    return ch
}

func GenerateInt() chan int {
    ch := make(chan int, 10)
    go func() {
        for {
            select {
                case ch <- <-GenerateIntA():
                case ch <- <-GenerateIntB():
            }
        }
    }()
    return ch
}

func main() {
    ch := GenerateInt()

    for i := 0; i < 100; i++ {
        fmt.Println(<-ch)
    }
}
```
- 借助 Go 通道的退出通知机制，实现生成器自动退出
``` demo-5.5
package main

import (
    "fmt"
    "math/rand"
)

func GenerateInt(done chan struct{}) chan int {
    ch := make(chan int)
    go func() {
        for {
            // 通过 select 监听一个信号 chan 来确定是否停止生成
            select {
                case ch <-rand.Ine():
                case <-done:
                    break
            }
        }
        close(ch)
    }()
    return ch
}

func main() {
    done := make(chan int)
    ch := GenerateInt(done)

    fmt.Println(<-ch)
    fmt.Println(<-ch)

    // 不再需要生成器，通过 close chan 发送一个通知给生成器
    close(done)
    for v := range ch {
        fmt.Println(v)
    }
}
```
- 融合并发、缓冲、退出机制的生成器
``` demo-5.6
package main

import (
    "fmt"
    "math/rand"
)

// done 接收通知退出信号
func GenerateIntA(done chan struct{}) chan int {
    ch := make(chan int, 10)
    go func() {
        for {
            select {
                case ch <-rand.Int():
                case <-done:
                    break
            }
        }
        close(ch)
    }()
    return ch
}

func GenerateIntB(done chan struct{}) chan int {
    ch := make(chan int, 10)
    go func() {
        for {
            select {
                case ch <-rand.Int():
                case <-done:
                    break
            }
        }
        close(ch)
    }()
    return ch
}

// 通过 select 执行扇入操作
func GenerateInt(done chan struct{}) chan int {
    ch := make(chan int)
    send := make(chan struct{})
    go func() {
        for {
            select {
                case ch <- <-GenerateIntA(send):
                case ch <- <-GenerateIntB(send):
                case <-done:
                    send <- struct{}{}
                    send <- struct{}{}
                    break
            }
        }
        close(ch)
    }()
    return ch
}

func main() {
    done := make(chan struct{})
    ch :=  GenerateInt(done)

    for i := 0; i < 100; i++ {
        fmt.Println(<-ch)
    }

    // 通知生产者停止生产
    done <-struct{}{}
    fmt.Prinln("stop generate")
}
```

**管道**  
通道可以分成两个方向，一个是读，另一个是写，假如一个函数的 输入参数和输出参数都是相同的 chan 类型，则该函数可以调用自己，最终形成一个调用链。多个具有相同参数类型的函数也能组成一个调用链，类似 UNIX 系统的管道，一个有类型的管道
``` demo-5.7
package main

import "fmt"

// chain 函数的输入输入参数类型相同，功能是将 chan 内的数据加一
func chain(in chan int) chan int {
    out := make(chan int)
    go func() {
        for v := range in {
            out <- v + 1
        }
        close(out)
    }()
    return out
}

func main() {
    in := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            in <- i
        }
        close(in)
    }()
    // 连续调用 3 次 chan，相当于把 in 中的每个元素都加 3
    out := chain(chain(chain(in)))
    for v := range out {
        fmt.Println(v)
    }
}
```

**每个请求一个 goroutine**  
来一个请求或任务就启动一个 goroutine 去处理，典型的就是 Go 中的 HTTP server 服务  
以计算 100 个自然数的和举例，将计算任务拆分为多个 task，每个 task 启动一个 goroutine 进行处理
``` demo-5.8
package main

import (
    "fmt"
    "sync"
)

// 工作任务
type task struct {
    begin int
    end int
    result chan<- int
}

// 任务执行：计算 begin 到 end 的和，将执行结果写入 result
func (t *task) do() {
    sum := 0
    for i := t.begin; i <= t.end; i++ {
        sum += i
    }
    t.result <- sum
}

func main() {
    // 创建任务通道
    taskchan := make(chan task, 10)
    // 创建结果通道
    resultchan := make(chan int, 10)
    // wg 用于同步等待任务的执行
    wg := &sync.WaitGourp{}
    // 初始化 task 的 goroutine，计算 100 个自然数之和
    go InitTask(taskchan, resultchan, 100)
    // 每个 task 启动一个 goroutine 进行处理
    go DistributeTask(taskchan, wg, resultchan)
    // 通过结果通道获取结果并汇总
    sum := ProcessResult(resultchan)
    fmt.Println("sum=", sum)
}

// 构建 task 并写入 task 通道
func InitTask(taskchan chan<- task, resultchan chan int, p int) {
    num := p / 10
    mod := p % 10
    for i := 0; i < num; i++ {
        b := 10 * i + 1
        e := 10 * (i + 1)
        t := task {
            begin: b,
            end: e,
            result: resultchan,
        }
        taskchan <- t
    }
    if mod != 0 {
        t := task {
            begin: num * 10 + 1,
            end: p,
            result: resultchan,
        }
        taskchan <- t
    }
    close(taskchan)
}

// 读取 task，每个 task 启动一个 worker goroutine 进行处理并等待每个 task 完成，关闭结果通道
func DistributeTask(taskchan <-chan task, wg *sync.WaitGroup, result chan int) {
    for v := range taskchan {
        wg.Add(1)
        go ProcessTask(v, wg)
    }
    wg.Wait()
    close(result)
}

// goroutine 处理具体工作，并将结果发送到结果通道
func ProcessTask(t task, wg *sync.WaitGroup) {
    t.do()
    wg.Done()
}

// 读取结果通道，汇总结果
func processResult(resultchan chan int) int {
    sum := 0
    for r := range resultchan {
        sum += r
    }
    return sum
}
```
> 程序逻辑分析  
> 1. InitTask 函数构建 task 并发送到 task 通道中
> 2. 分发任务函数 DistributeTask 为每个 task 启动一个 goroutine 处理任务，等待其处理完成，然后关闭结果通道
> 3. ProcessResult 函数读取并统计所有的结果
> 
> 函数在不同的 goroutine 中运行，他们通过通道和 sync.WaitGroup 进行通信和同步

**固定 worker 工作池**  
服务器编程中使用最多的是通过线程池来提升服务的并发处理能力。在 Go 语言编程中一样可以轻松构建固定数量的 goroutine 作为工作线程池  
以计算多个整数的和为例  
除 main goroutine 外，还有以下几类 goroutine
1. 初始化任务的 goroutine
2. 分发任务的 goroutine
3. 等待所有 worker 结束通知，然后关闭结果通道的 goroutine

采用的通道
1. 传递 task 任务的通道
2. 传递 task 结果的通道
3. 接收 worker 处理完任务后所发通知的通道
``` demo-5.9
package main

import "fmt"

// 工作池的 goroutine 数目
const NUMBER = 10

// 工作任务
type task struct {
    begin  int
    end    int
    result chan<- int
}

// 任务处理： 计算 begin 到 end 的和，将结果写入 chan result
func (t *task)Do() {
    sum := 0
    for i := t.begin; i <= t.end; i++ {
        sum += i
    }
    t.result <- sum
}

func main() {
    workers := NUMBER
    // 工作通道
    taskChan := make(chan task, 10)
    // 结果通道
    resultChan := make(chan int, 10)
    // worker 信号通道
    done := make(chan struct{}, 10)
    
    // 初始化 task 的 goroutine，计算 100 个自然数和
    go InitTask(taskChan, resultChan, 100)

    // 分发任务到 NUMBER 个 goroutine 池
    DistributeTask(taskChan, workers, done)

    // 获取各个 goroutine 处理完任务的通知并关闭结果通道
    go CloseResult(done, resultChan, workers)

    // 通过结果通道获取结果并汇总
    sum := ProcessResult(resultChan)

    fmt.Println("sum=", sum)
}

// 初始化待处理的 task chan
func InitTask(taskChan chan<- task, resultChann chan int, p int) {
    num := p / 10
    mod := p % 10
    for i := 0; i < num; i++ {
        tsk := task{
            begin:  10 * i + 1,
            end:    10 * (i + 1),
            result: resultChan,
        }
        taskChan <- tsk
    }
    if mod != 0 {
        tsk := task{
            begin:  10 * num + 1,
            end:    p,
            result: resultChan,
        }
        taskChan <- tsk
    }
    close(taskChan)
}

// 读取 task chan 并分发到 worker goroutine 处理，总的数量是 workers
func DistributeTask(taskChan <-chan task, workers int, done chan struct{}) {
    for i := 0; i < workers; i++ {
        go ProcessTask(taskChan, done)
    }
}

// 工作 goroutine 处理具体工作并将处理结果发送到结果 chan
func ProcessTask(taskChan <-chan task, done chan struct{}) {
    for task := range taskChan {
        t.do()
    }
    done <- struct{}{}
}

// 通过 done channel 同步等待所有工作 goroutine 的结束，然后关闭结果 chan
func CloseResult(done chan struct{}, resultChan chan int, workers int) {
    for i := 0; i < workers; i++ {
        <-done
    }
    close(done)
    close(resultChan)
}

// 读取结果通道，汇总结果
func ProcessResult(resultChan chan int) int {
    sum := 0
    for result := range resultChan {
        sum += result
    }
    return sum
}
```
> 程序逻辑分析：
> 1. 构建 task 并发送到 task 通道中
> 2. 分别启动 n 个工作线程，不停地从 task 通道获取任务，然后将结果写入结果通道。如果任务通道被关闭，则负责向收敛结果的 goroutine 发送通知，告诉其当前 worker 已经完成工作
> 3. 收敛结果的 goroutine 接收到所有 task 已经处理完毕的信号后，主动关闭结果通道
> 4. main 中的函数 ProcessResult 读取并统计所有结果

**future 模式**  
在一个流程中需要调用多个子调用，而这些子调用相互之间没有依赖，如果串行地调用，则耗时会很长。此时可以使用 Go 并发编程中的 future 模式  
future 模式的基本工作原理
1. 使用 chan 作为函数参数
2. 启动 goroutine 调用函数
3. 通过 chan 传入参数
4. 同时并行无依赖的子调用
5. 通过 chan 异步获取结果
``` demo-5.10
package main

import (
    "fmt"
    "time"
)

// 异步查询数据库的例子
// sql 查询参数，result 返回结果
type Query struct {
    // 简单的抽象，实际更复杂
    sql chan string
    result chan string
}

func main() {
    // 初始化 Query
    query := Query{
        sql:    make(chan string, 1),
        result: make(chan string, 1),
    }

    // 执行 Query，无须准备参数
    go executeQuery(query)

    // 传递参数
    query.sql <- "select * from table"

    // 做其他的事情，假设花费 1 秒钟时间
    time.Sleep(1 * time.Second)

    // 获取数据库取回的结果
    fmt.Println(<-query.result)
}

func executeQuery(query Query) {
    for sql := range query.sql {
        // 访问数据库的操作
        query.result <- "result from " + sql
    }
}
```

### context 标准库
Go 中的 goroutine 之间没有父子关系，也就没有子进程退出后的通知机制，多个 goroutine 是平行的被调度，多个 goroutine 如何协作工作涉及通信、同步、通知和退出四个方面
- 通信：chan 通道是 goroutine 之间通信的基础，通信主要指程序的数据通道
- 同步：不带缓冲的 chan 提供了一个天然的同步等待机制，sync.WaitGroup 也为多个 goroutine 协同工作提供了一种同步等待机制
- 通知：通知和通信的数据不同，通知通常不是业务数据，而是管理、控制流数据。可以在输入端绑定两个 chan，一个用于业务流数据，另一个用于异常通知数据，然后通过 select 收敛进行处理。但不是一个通用的解决方案
- 退出：goroutine 之间没有父子关系，可以通过增加一个单独的通道，借助通道和 select 的广播机制（close channel to broadcast）实现退出

**context 的设计目的**  
context 库的设计目的就是跟踪 goroutine 调用树，并在这些 goroutine 调用树中传递通知和元数据
- 退出通知机制：通知可以传递给整个 goroutine 调用树上的每一个 goroutine
- 传递数据：数据可以传递给整个 goroutine 调用树上的每一个 goroutine

**基本数据结构**  
context 包的整体工作机制：第一个创建 Context 的 goroutine 被称为 root 节点。root 节点负责创建一个实现 Context 接口的具体对象，并将该对象作为参数传递到其新拉起的 goroutine，下游的 goroutine 可以继续封装该对象，在传递到更下游的 goroutine。Context 对象在传递的过程中最终形成一个树状的数据结构，通过位于 root 节点（树的根节点）的 Context 对象就能遍历整个 Context 对象树。通知和消息可以通过 root 节点传递出去，实现上游 goroutine 对下游 goroutine 的消息传递

- Context 接口
> Context 是一个基本接口，所有的 Context 对象都要实现该接口，context 的使用者在调用接口中都是用 Context作为参数类型
``` demo-5.11
type Context interface {
    // 如果 Context 实现了超时控制，则该方法返回 ok true，deadline 为超时时间。否则 ok 为 false
    Deadline() (deadline time.Time, ok bool)

    // 后端被调的 goroutine 应该监听该方法返回的 chan，以便及时释放资源
    Done() <-chan struct{}

    // Done 返回的 chan 收到通知的时候，才可以访问 Err() 获知因为什么原因被取消
    Err() error

    // 可以访问上游 goroutine 传递给下游 goroutine 的值
    Value(key interface{}) interface{}
}
```
- canceler 接口
> canceler 接口是一个扩展接口，规定了取消通知的 Context 具体类型需要实现的接口。context 包中的具体类型 *cancelCtx 和 *timerCtx 都实现了该接口
``` demo-5.12
// context 对象如果实现了 canceler 接口，则可以被取消
type canceler interface {
    // 创建 cancel 接口实例的 goroutine 调用 cancel 方法通知后续创建的 goroutine 退出
    cancel(removeFromParent bool, err error)

    // Done 方法返回的 chan 需要后端 goroutine 来监听，并及时退出
    Done() <-chan struct{}
}
```
- empty Context 结构
> emptyCtx 实现了 Context 接口，但不具备任何功能，因为其所有的方法都是空实现。其存在的目的是作为 Context 对象树的根（root 节点）

