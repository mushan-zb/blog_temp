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
    //强制类型转换
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
**注：** 使用 type 定义的新类型不会继承旧类型的方法

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

## 接口
