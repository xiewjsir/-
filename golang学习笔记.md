##### iota枚举
```
import "fmt"

func main()  {
	const (
		a = iota
		b = iota
		c
		d
	)
	fmt.Println(a,b,c,d)//0 1 2 3

	const e = iota
	fmt.Println(e)//0

	const (
		f,g,h = iota,iota,iota
	)
	fmt.Println(f,g,h)//0 0 0

	const(
		i = iota
		j = "b"
		k
		z  = iota
		l,n,m = iota,iota,iota
		o = iota
	)
	fmt.Println(i,j,k,z,l,n,m,o)//0 b b 3 4 4 4 5
}
```
##### 基础数据类型

类型 | 名称 | 长度 | 零值 | 说明
---|---|---|---|---
bool | 布尔类型 | 1 | false | 其值不为真即为家，不可以用数字代表true或false
byte | 字节型 | 1 | 0 | uint8别名
rune | 字符类型 | 4 | 0 | 专用于存储unicode编码，等价于uint32
int, uint | 整型 | 4或8 | 0 | 32位或64位
int8, uint8 | 整型 | 1 | 0 | -128 ~ 127, 0 ~ 255
int16, uint16 | 整型 | 2 | 0 | -32768 ~ 32767, 0 ~ 65535
int32, uint32 | 整型 | 4 | 0 |-21亿 ~ 21 亿, 0 ~ 42 亿
int64, uint64 | 整型 | 8 | 0 |
float32 | 浮点型 | 4 | 0.0 | 小数位精确到7位
float64 | 浮点型 | 8 | 0.0 | 小数位精确到15
complex64 | 复数类型 | 8 |  | 
complex128 | 复数类型 | 16 | | 
uintptr | 整型 | 4或8 | false |⾜以存储指针的uint32或uint64整数
string | 字符串 |  |  | utf-8字符串

##### fmt包的格式化输出输入
###### 格式符
格式 | 含义 
---|---
string | 字符串
%% | 一个%字面量
%b | 一个二进制整数值(基数为2)，或者是一个(高级的)用科学计数法表示的指数为2的浮点数
%c | 字符型。可以把输入的数字按照ASCII码相应转换为对应的字符
%d | 一个十进制数值(基数为10)
%e | 以科学记数法e表示的浮点数或者复数值
%E | 以科学记数法E表示的浮点数或者复数值
%f | 以标准记数法表示的浮点数或者复数值
%g | 以%e或者%f表示的浮点数或者复数，任何一个都以最为紧凑的方式输出
%G | 以%E或者%f表示的浮点数或者复数，任何一个都以最为紧凑的方式输出
%o | 一个以八进制表示的数字(基数为8)
%p | 以十六进制(基数为16)表示的一个值的地址，前缀为0x,字母使用小写的a-f表示
%q | 使用Go语法以及必须时使用转义，以双引号括起来的字符串或者字节切片[]byte，或者是以单引号括起来的数字
%s | 字符串。输出字符串中的字符直至字符串中的空字符（字符串以'\0‘结尾，这个'\0'即空字符）
%t | 以true或者false输出的布尔值
%T | 使用Go语法输出的值的类型
%U | 一个用Unicode表示法表示的整型码点，默认值为4个数字字符
%v | 使用默认格式输出的内置或者自定义类型的值，或者是使用其类型的String()方式输出的自定义值，如果该方法存在的话
%x | 以十六进制表示的整型值(基数为十六)，数字a-f使用小写表示
%X | 以十六进制表示的整型值(基数为十六)，数字A-F使用小写表示


##### new 函数
> 表达式new(T)将创建一个T类型的匿名变量，所做的是为T类型的新值分配并清零一块内存空间，然后将这块内存空间的地址作为结果返回，而这个结果就是指向这个新的T类型值的指针值，返回的指针类型为*T。
```
func main() {
    var p1 *int
    p1 = new(int)              //p1为*int 类型, 指向匿名的int变量
    fmt.Println("*p1 = ", *p1) //*p1 =  0

    p2 := new(int) //p2为*int 类型, 指向匿名的int变量
    *p2 = 111
    fmt.Println("*p2 = ", *p2) //*p1 =  111
}

```

```
func swap01(a, b int) {
    a, b = b, a
    fmt.Printf("swap01 a = %d, b = %d\n", a, b)
}

func swap02(x, y *int) {
    *x, *y = *y, *x
}

func main() {
    a := 10
    b := 20

    //swap01(a, b) //值传递
    swap02(&a, &b) //变量地址传递
    fmt.Printf("a = %d, b = %d\n", a, b)
}

```
##### 数组
```
    a := [3]int{1, 2}           // 未初始化元素值为 0
    b := [...]int{1, 2, 3}      // 通过初始化值确定数组长度
    c := [5]int{2: 100, 4: 200} // 通过索引号初始化元素，未初始化元素值为 0
    fmt.Println(a, b, c)        //[1 2 0] [1 2 3] [0 0 100 0 200]

    //支持多维数组
    d := [4][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}}
    e := [...][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}} //第二维不能写"..."
    f := [4][2]int{1: {20, 21}, 3: {40, 41}}
    g := [4][2]int{1: {0: 20}, 3: {1: 41}}
    
    fmt.Println(d, e, f, g)
```
##### 切片
数组的长度在定义之后无法再次修改；数组是值类型，每次传递都将产生一份副本。显然这种数据结构无法完全满足开发者的真实需求。Go语言提供了数组切片（slice）来弥补数组的不足。

切片并不是数组或数组指针，它通过内部指针和相关属性引⽤数组⽚段，以实现变⻓⽅案。

slice并不是真正意义上的动态数组，而是一个引用类型。slice总是指向一个底层array，slice的声明也可以像array一样，只是不需要长度
。

**slice和数组的区别：** 声明数组时，方括号内写明了数组的长度或使用...自动计算长度，而声明slice时，方括号内没有任何字符。

**注意：** make只能创建slice、map和channel，并且返回一个有初始值(非零)。new 返回一个指针
```
var s1 []int //声明切片和声明array一样，只是少了长度，此为空(nil)切片
    s2 := []int{}

    //make([]T, length, capacity) //capacity省略，则和length的值相同
var s1 []int //声明切片和声明array一样，只是少了长度，此为空(nil)切片
var s3 []int = make([]int, 0)
s4 := make([]int, 0, 0)

s5 := []int{1, 2, 3} //创建切片并初始化

操作	            含义
s[n]	            切片s中索引位置为n的项
s[:]	            从切片s的索引位置0到len(s)-1处所获得的切片
s[low:]	            从切片s的索引位置low到len(s)-1处所获得的切片
s[:high]	        从切片s的索引位置0到high处所获得的切片，len=high
s[low:high]	        从切片s的索引位置low到high处所获得的切片，len=high-low
s[low:high:max]	    从切片s的索引位置low到high处所获得的切片，len=high-low，cap=max-low
len(s)	            切片s的长度，总是<=cap(s)
cap(s)	            切片s的容量，总是>=len(s)

var var1 [16]byte //array 数组
var1 = [16]byte{'a','b','a','b','a','b','a','b','a','b','a','b','a','b','a','b'}

var2 := make([]byte,16) //slice 切片
var2 = []byte("123450")
uuid.NewV3(var1,"cc")
fmt.Println(var1)
fmt.Println(var2)
```
##### map
>Go语言中的map(映射、字典)是一种内置的数据结构，它是一个无序的key—value对的集合，比如以身份证号作为唯一键来标识一个人的信息。

- map格式为
```
map[keyType]valueType
```
- map的创建
```
var m1 map[int]string  //只是声明一个map，没有初始化, 此为空(nil)map
fmt.Println(m1 == nil) //true
//m1[1] = "mike" //err, panic: assignment to entry in nil map

//m2, m3的创建方法是等价的
m2 := map[int]string{}
m3 := make(map[int]string)
fmt.Println(m2, m3) //map[] map[]

m4 := make(map[int]string, 10) //第2个参数指定容量
fmt.Println(m4)                //map[]

```
- map做函数参数
```
#在函数间传递映射并不会制造出该映射的一个副本，不是值传递，而是引用传递：

func DeleteMap(m map[int]string, key int) {
    delete(m, key) //删除key值为2的map

    for k, v := range m {
        fmt.Printf("len(m)=%d, %d ----> %s\n", len(m), k, v)
        //len(m)=2, 1 ----> mike
        //len(m)=2, 3 ----> lily
    }
}

func main() {
    m := map[int]string{1: "mike", 2: "yoyo", 3: "lily"}

    DeleteMap(m, 2) //删除key值为2的map

    for k, v := range m {
        fmt.Printf("len(m)=%d, %d ----> %s\n", len(m), k, v)
        //len(m)=2, 1 ----> mike
        //len(m)=2, 3 ----> lily
    }
}

```
##### 结构体
- 结构体初始化
```
#普通变量
type Student struct {
    id   int
    name string
    sex  byte
    age  int
    addr string
}

func main() {
    //1、顺序初始化，必须每个成员都初始化
    var s1 Student = Student{1, "mike", 'm', 18, "sz"}
    s2 := Student{2, "yoyo", 'f', 20, "sz"}
    //s3 := Student{2, "tom", 'm', 20} //err, too few values in struct initializer

    //2、指定初始化某个成员，没有初始化的成员为零值
    s4 := Student{id: 2, name: "lily"}
}

#指针变量
type Student struct {
    id   int
    name string
    sex  byte
    age  int
    addr string
}

func main() {
    var s5 *Student = &Student{3, "xiaoming", 'm', 16, "bj"}
    s6 := &Student{4, "rocco", 'm', 3, "sh"}
}
```
##### 不定参数类型
不定参数是指函数传入的参数个数为不定数量。为了做到这点，首先需要将函数定义为接受不定参数类型：
```
//形如...type格式的类型只能作为函数的参数类型存在，并且必须是最后一个参数
func MyFunc01(args ...int) {
    fmt.Println("MyFunc01")
    for _, n := range args { //遍历参数列表
        fmt.Println(n)
    }
}

func MyFunc02(args ...int) {
    fmt.Println("MyFunc02")
    for _, n := range args { //遍历参数列表
        fmt.Println(n)
    }
}

func Test(args ...int) {
    MyFunc01(args...)     //按原样传递, Test()的参数原封不动传递给MyFunc01
    MyFunc02(args[1:]...) //Test()参数列表中，第1个参数及以后的参数传递给MyFunc02
}

func main() {
    Test(1, 2, 3) //函数调用
}

```
```

```
##### 方法
>在Go语言中，可以给任意自定义类型（包括内置类型，但不包括指针类型）添加相应的方法。
⽅法总是绑定对象实例，并隐式将实例作为第⼀实参 (receiver)，方法的语法如下：
```
func (receiver ReceiverType) funcName(parameters) (results)
```
- 参数 receiver 可任意命名。如⽅法中未曾使⽤，可省略参数名。
- 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接⼝或指针。
- 不支持重载方法，也就是说，不能定义名字相同但是不同参数的方法。

###### 基础类型作为接收者
>通过下面的例子可以看出，面向对象只是换了一种语法形式来表达。方法是函数的语法糖，因为receiver其实就是方法所接收的第1个参数。注意：虽然方法的名字一模一样，但是如果接收者不一样，那么方法就不一样。

```
type MyInt int //自定义类型，给int改名为MyInt

//在函数定义时，在其名字之前放上一个变量，即是一个方法
func (a MyInt) Add(b MyInt) MyInt { //面向对象
    return a + b
}

//传统方式的定义
func Add(a, b MyInt) MyInt { //面向过程
    return a + b
}

func main() {
    var a MyInt = 1
    var b MyInt = 1

    //调用func (a MyInt) Add(b MyInt)
    fmt.Println("a.Add(b) = ", a.Add(b)) //a.Add(b) =  2

    //调用func Add(a, b MyInt)
    fmt.Println("Add(a, b) = ", Add(a, b)) //Add(a, b) =  2
}

```
###### 结构体作为接收者
```
type Person struct {
    name string
    sex  byte
    age  int
}

func (p Person) PrintInfo() { //给Person添加方法
    fmt.Println(p.name, p.sex, p.age)
}

func main() {
    p := Person{"mike", 'm', 18} //初始化
    p.PrintInfo() //调用func (p Person) PrintInfo()
}

```
###### 类型 *T 方法集
>一个指向自定义类型的值的指针，它的方法集由该类型定义的所有方法组成，无论这些方法接受的是一个值还是一个指针。
如果在指针上调用一个接受值的方法，Go语言会聪明地将该指针解引用，并将指针所指的底层值作为方法的接收者。
```
package main

import "fmt"

type Person struct {
	name string
	sex byte
	age int
}

//指针作为接收者，引用语义
func (p *Person) SetInfoPointer(){
	(* p).name = "yoyo"
	p.sex = 'f'
	p.age = 18
}

func (p Person) SetInfoValue(){
	p.name = "xixi"
	p.sex = 'm'
	p.age = 22
}

func main(){
	var p *Person = &Person{"mike",'m',20}
	fmt.Println(*p) //{mike 109 20}

	p.SetInfoPointer()
	fmt.Println(*p) //{yoyo 102 18}

	p.SetInfoValue()
	fmt.Println(*p) //{yoyo 102 18}

	(*p).SetInfoValue()
	fmt.Println(*p) //{yoyo 102 18}
}
```
###### 类型 T 方法集
> 一个自定义类型值的方法集则由为该类型定义的接收者类型为值类型的方法组成，但是不包含那些接收者类型为指针的方法。
但这种限制通常并不像这里所说的那样，因为如果我们只有一个值，仍然可以调用一个接收者为指针类型的方法，这可以借助于Go语言传值的地址能力实现。

```
type Person struct {
    name string
    sex  byte
    age  int
}

//指针作为接收者，引用语义
func (p *Person) SetInfoPointer() {
    (*p).name = "yoyo"
    p.sex = 'f'
    p.age = 22
}
//值作为接收者，值语义
func (p Person) SetInfoValue() {
    p.name = "xxx"
    p.sex = 'm'
    p.age = 33
}

func main() {
	//p 为普通值类型
	var p Person = Person{"mike", 'm', 18}
	fmt.Println(p) //{mike 109 18}

	(&p).SetInfoPointer() //func (&p) SetInfoPointer()
	fmt.Println(p) //{yoyo 102 22}

	(&p).SetInfoValue()   //func (*&p) SetInfoValue()
	fmt.Println(p) //{yoyo 102 22}

	p = Person{"mike",'m',18}
	p.SetInfoPointer()    //func (&p) SetInfoPointer()
	fmt.Println(p) //{yoyo 102 22}

	p.SetInfoValue()      //func (p) SetInfoValue()
	fmt.Println(p) //{yoyo 102 22}
}

```

##### 匿名字段
##### 方法的继承
```
type Person struct {
	name string
	sex byte
	age int
}

func (p *Person) Printinfo(){
	fmt.Printf("%s,%c,%d\n",p.name,p.sex,p.age)
}

type Student struct {
	Person
	id	int
	addr string
}

func main(){
	p := Person{"mike",'m',18}
	p.Printinfo()

	s := Student{Person{"yoyo",'f',20},2,"sz"}
	s.Printinfo()
}
```
###### 方法的重写
```
type Person struct{
	name string
	sex byte
	age int
}

//Person定义了方法
func (p *Person) PrintInfo(){
	fmt.Printf("Person:%s,%c,%d\n",p.name,p.sex,p.age)
}

type Student struct {
	Person
	id int
	addr string
}

func (s *Student) PrintInfo(){
	fmt.Printf("Student：%s,%c,%d\n", s.name, s.sex, s.age)
}

func main()  {
	p := Person{"mike", 'm', 18}
	p.PrintInfo() //Person: mike,m,18

	s := Student{Person{"yoyo", 'f', 20}, 2, "sz"}
	s.PrintInfo()        //Student：yoyo,f,20
	s.Person.PrintInfo() //Person: yoyo,f,20
}
```
##### 表达式
类似于我们可以对函数进行赋值和传递一样，方法也可以进行赋值和传递。
根据调用者不同，方法分为两种表现形式：方法值和方法表达式。两者都可像普通函数那样赋值和传参，区别在于方法值绑定实例，⽽方法表达式则须显式传参。
```
type Person struct {
    name string
    sex  byte
    age  int
}

func (p *Person) PrintInfoPointer() {
    fmt.Printf("%p, %v\n", p, p)
}

func (p Person) PrintInfoValue() {
    fmt.Printf("%p, %v\n", &p, p)
}

func main(){
	p := Person{"mike",'m',20}
	p.PrintInfoPointer() //0xc000004440,&{mike 109 20}

	//方法值，隐式传递 receiver
	pFunc1 := p.PrintInfoPointer
	pFunc1() //0xc000004440,&{mike 109 20}

	pFunc2 := p.PrintInfoValue
	pFunc2() //0xc0000044c0,{mike 109 20}

	//方法表达式， 须显式传参
	//func pFunc1(p *Person))
	pFunc3 := (*Person).PrintInfoPointer
	pFunc3(&p) //0xc000004440,&{mike 109 20}

	pFunc4 := Person.PrintInfoValue
	pFunc4(p) //0xc000004520,{mike 109 20}

}

```
##### 接口
> 接口是用来定义行为的类型。这些被定义的行为不由接口直接实现，而是通过方法由用户定义的类型实现，一个实现了这些方法的具体类型是这个接口类型的实例。
```
type Humaner interface {
	SayHi()
}

type Student struct {//学生
	name string
	score float64
}

func (s *Student) SayHi(){
	fmt.Printf("Student[%s,%f] say hi!\n",s.name,s.score)
}

type Teacher struct {
	name string
	group string
}

func (t *Teacher) SayHi(){
	fmt.Printf("Teacher[%s,%s] say hi!\n",t.name,t.group)
}

type MyStr string

func (str MyStr) SayHi(){
	fmt.Printf("MyStr[%s] say hi!\n\n",str)
}

func WhoSayHi(i Humaner){
	i.SayHi()
}

func main(){
	s := &Student{"mike",88.88}
	t := &Teacher{"yoyo","Go 语言"}
	var tmp MyStr = "测试"

	s.SayHi()
	t.SayHi()
	tmp.SayHi()

	//多态，调用同一接口，不同表现
	WhoSayHi(s)
	WhoSayHi(t)
	WhoSayHi(tmp)

	x := make([]Humaner,3)

	//这三个都是不同类型的元素，但是他们实现了interface同一个接口
	x[0],x[1],x[2] = s,t,tmp
	for _,value := range x{
		value.SayHi()
	}
}
```
##### 空接口
>open the door短片 智能系统
```
var v1 interface{} = 1 // 将int类型赋值给interface{}
var v2 interface{} = "abc"  // 将string类型赋值给interface{}
var v3 interface{} = &v2    // 将*interface{}类型赋值给interface{}
var v4 interface{} = struct {
	x int
}{1}
var v5 interface{} = &struct {
	x int
}{2}

fmt.Printf("%d,%s,%p,%v,%p",v1,v2,v3,v4,v5)
//1,abc,0xc0000441e0,{1},0xc00000a100
```
当函数可以接受任意的对象实例时，我们会将其声明为interface{}，最典型的例子是标准库fmt中PrintXXX系列的函数，例如：
```
func Printf(fmt string, args ...interface{})
func Println(args ...interface{})
```
##### 类型查询
>我们知道interface的变量里面可以存储任意类型的数值(该类型实现了interface)。那么我们怎么反向知道这个变量里面实际保存了的是哪个类型的对象呢？目前常用的有两种方法：
- comma-ok断言
- switch测试

```
type Element interface {}

type Person struct{
	name string
	age int
}

func main() {
	list := make([]Element,3)
	list[0] = 1
	list[1] = "Hello"
	list[2] = Person{"mike",20}
    //comma-ok断言
	for index,element := range list{
		if value,ok := element.(int);ok{
			fmt.Printf("list[%d] is an int and its value is %d\n",index,value)
		}else if value,ok := element.(string);ok{
			fmt.Printf("list[%d] is an string and its value is %s\n",index,value )
		}else if value,ok := element.(Person);ok{
			fmt.Printf("list[%d] is a Person and its value is [%s,%d]\n",index,value.name,value.age);
		}else{
			fmt.Printf("list[%d] is of a different type\n",index)
		}
	}
    //switch测试
	for index, element := range list {
		switch value := element.(type) {
		case int:
			fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
		case string:
			fmt.Printf("list[%d] is an string and its value is %s\n", index, value)
		case Person:
			fmt.Printf("list[%d] is a Person and its value is [%s,%d]\n", index, value.name, value.age);
		default:
			fmt.Printf("list[%d] is of a different type\n", index)
		}
	}	
}

//list[0] is an int and its value is 1
//list[1] is an string and its value is Hello
//list[2] is a Person and its value is [mike,20]
//list[0] is an int and its value is 1
//list[1] is an string and its value is Hello
//list[2] is a Person and its value is [mike,20]
```
##### 异常处理
###### error接口
> Go语言引入了一个关于错误处理的标准模式，即error接口，它是Go语言内建的接口类型，该接口的定义如下
```
type error interface {
    Error() string
}
```
```
func main() {
	var err1 error = errors.New("a normal err1")
	fmt.Println(err1) //a normal err1

	var err2 error = fmt.Errorf("%s", "a normal err2")
	fmt.Println(err2) //a normal err2
}
```
###### panic
###### recover
>recover只有在defer调用的函数中有效。

##### JSON处理
```
type IT struct {
	Company  string
	Subjects []string
	IsOk     bool
	Price    float64
}

type Person struct {
	name string
	Sex byte
}

func main() {
	t1 := IT{"itcast",[]string{"GO","C++","Python"},true,87.66}

	b,err := json.MarshalIndent(t1,"","	")
	if err != nil{
		fmt.Println("json error:",err)
	}

	fmt.Println(string(b))

	b2,_ := json.Marshal(t1)
	fmt.Println(string(b2))

	t2 := Person{"mike",'m'}
	b3,_ := json.MarshalIndent(t2,"","	")
	fmt.Println(string(b3))
}

/*
{
	"Company": "itcast",
	"Subjects": [
		"GO",
		"C++",
		"Python"
	],
	"IsOk": true,
	"Price": 87.66
}

{"Company":"itcast","Subjects":["GO","C++","Python"],"IsOk":true,"Price":87.66}

{
	"Sex": 109
}
*/
```
我们看到上面的输出字段名的首字母都是大写的，如果你想用小写的首字母怎么办呢？把结构体的字段名改成首字母小写的？JSON输出的时候必须注意，只有导出的字段(首字母是大写)才会被输出，如果修改字段名，那么就会发现什么都不会输出，所以必须通过struct tag定义来实现。

针对JSON的输出，我们在定义struct tag的时候需要注意的几点是：
- 字段的tag是"-"，那么这个字段不会输出到JSON
- tag中带有自定义名称，那么这个自定义名称会出现在JSON的字段名中
- tag中如果带有"omitempty"选项，那么如果该字段值为空，就不会输出到JSON串中
- 如果字段类型是bool, string, int,int64等，而tag中带有",string"选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串
```
type IT struct {
	//Company不会导出到JSON中
	Company string `json:"_"`

	// Subjects 的值会进行二次JSON编码
	Subjects []string `json:"subjects"`

	//转换为字符串，再输出
	IsOk bool `json:",string"`
	
	// 如果 Price 为空，则不输出到JSON串中
	Price float64 `json:"price,omitempty"`
	
	Num int `json:"num,string"`

	Name string
}

func main()  {
	t1 := IT{"itcast",[]string{"Go","C++","Python","Test"},true,88.96,99,"mike"}
	b,err := json.MarshalIndent(t1,"","	")
	if err != nil{
		fmt.Println("json err:",err)
	}

	fmt.Println(string(b))
}
/*
{
	"_": "itcast",
	"subjects": [
		"Go",
		"C++",
		"Python",
		"Test"
	],
	"IsOk": "true",
	"price": 88.96,
	"num": "99",
	"Name": "mike"
}
*/
```
###### 通过map生成JSON
```
func main() {
	t1 := make(map[string]interface{})
	t1["company"] = "itcast"
	t1["subjects"] = []string{"Go","Python","C++","Test"}
	t1["isok"] = true
	t1["price"] = 66.89

	b,err := json.Marshal(t1)
	if err != nil{
		fmt.Println("json error:",err)
	}

	fmt.Println(string(b))
}
//{"company":"itcast","isok":true,"price":66.89,"subjects":["Go","Python","C++","Test"]}
```
#####  并发编程
- 并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。
- 并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。
- 并行是两个队列同时使用两台咖啡机
- 并发是两个队列交替使用一台咖啡机
###### goroutine是什么
> goroutine是Go并行设计的核心。goroutine说到底其实就是协程，但是它比线程更小，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。执行goroutine只需极少的栈内存(大概是4~5KB)，当然会根据相应的数据伸缩。也正因为如此，可同时运行成千上万个并发任务。goroutine比thread更易用、更高效、更轻便。
###### Gosched
> runtime.Gosched() 用于让出CPU时间片，让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行。
这就像跑接力赛，A跑了一会碰到代码runtime.Gosched() 就把接力棒交给B了，A歇着了，B继续跑。
```
func main() {
	go func(s string) {
		for i := 0; i < 99; i++ {
			fmt.Println(s)
		}
	}("Hello go")

	for i := 0; i < 2; i++ {
		runtime.Gosched()
		fmt.Println("hello")
	}
}
//在64位window10系统测试，输出结果不可控，64位ubuntu系统测试按预计顺序输出。
```
###### Goexit 
>调用 runtime.Goexit() 将立即终止当前 goroutine 执⾏，调度器确保所有已注册 defer延迟调用被执行。
```
func main()  {
    go func() {
        defer fmt.Println("A.defer")

        func() {
            defer fmt.Println("B.defer")
            runtime.Goexit() // 终止当前 goroutine, import "runtime"
            fmt.Println("B") // 不会执行
        }()

        fmt.Println("A") // 不会执行
    }() //别忘了()

    //死循环，目的不让主goroutine结束
    for {
    }

}
```
###### GOMAXPROCS
>调用 runtime.GOMAXPROCS() 用来设置可以并行计算的CPU核数的最大值，并返回之前的值。
```
func main() {
    //n := runtime.GOMAXPROCS(1) //打印结果：111111111111111111110000000000000000000011111...
    n := runtime.GOMAXPROCS(2)     //打印结果：010101010101010101011001100101011010010100110...
    fmt.Printf("n = %d\n", n)

    for {
        go fmt.Print(0)
        fmt.Print(1)
    }
}
//以上为教程测试结果，实测车出顺序不可控（4核CPU,WINDOW10,UBUNTU）
```
##### channel
>goroutine运行在相同的地址空间，因此访问共享内存必须做好同步。goroutine 奉行通过通信来共享内存，而不是共享内存来通信。
引⽤类型 channel 是 CSP 模式的具体实现，用于多个 goroutine 通讯。其内部实现了同步，确保并发安全。
```
make(chan Type) //等价于make(chan Type, 0)
make(chan Type, capacity)
/*
当 capacity= 0 时，channel 是无缓冲阻塞读写的，当capacity> 0 时，channel有缓冲、是非阻塞的，直到写满 capacity个元素才阻塞写入
*/

channel <- value      //发送value到channel
<-channel             //接收并将其丢弃
x := <-channel        //从channel中接收数据，并赋值给x
x, ok := <-channel    //功能同上，同时检查通道是否已关闭或者是否为空
```
###### 无缓冲的channel
无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。
这种类型的通道要求发送 goroutine 和接收 goroutine 同时准备好，才能完成发送和接收操作。如果两个goroutine没有同时准备好，通道会导致先执行发送或接收操作的 goroutine 阻塞等待。
这种对通道进行发送和接收的交互行为本身就是同步的。其中任意一个操作都无法离开另一个操作单独存在。

###### 有缓冲的channel
有缓冲的通道（buffered channel）是一种在被接收前能存储一个或者多个值的通道。
这种类型的通道并不强制要求 goroutine 之间必须同时完成发送和接收。通道会阻塞发送和接收动作的条件也会不同。只有在通道中没有要接收的值时，接收动作才会阻塞。只有在通道没有可用缓冲区容纳被发送的值时，发送动作才会阻塞。
这导致有缓冲的通道和无缓冲的通道之间的一个很大的不同：无缓冲的通道保证进行发送和接收的 goroutine 会在同一时间进行数据交换；有缓冲的通道没有这种保证。

###### 单方向的channel
默认情况下，通道是双向的，也就是，既可以往里面发送数据也可以同里面接收数据。
但是，我们经常见一个通道作为参数进行传递而值希望对方是单向使用的，要么只让它发送数据，要么只让它接收数据，这时候我们可以指定通道的方向。
单向channel变量的声明非常简单，如下：
```
var ch1 chan int       // ch1是一个正常的channel，不是单向的
var ch2 chan<- float64 // ch2是单向channel，只用于写float64数据
var ch3 <-chan int     // ch3是单向channel，只用于读取int数据

// chan<- 表示数据进入管道，要把数据写进管道，对于调用者就是输出。
// <-chan 表示数据从管道出来，对于调用者就是得到管道的数据，当然就是输入。

```
##### 定时器
###### Timer
Timer是一个定时器，代表未来的一个单一事件，你可以告诉timer你要等待多长时间，它提供一个channel，在将来的那个时间那个channel提供了一个时间值。

###### Ticker
Ticker是一个定时触发的计时器，它会以一个间隔(interval)往channel发送一个事件(当前时间)，而channel的接收者可以以固定的时间间隔从channel中读取事件。

###### select
Go里面提供了一个关键字select，通过select可以监听channel上的数据流动。
select的用法与switch语言非常类似，由select开始一个新的选择块，每个选择条件由case语句来描述。
与switch语句可以选择任何可使用相等比较的条件相比， select有比较多的限制，其中最大的一条限制就是每个case语句里必须是一个IO操作，大致的结构如下：
```
select {
    case <-chan1:
        // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
        // 如果成功向chan2写入数据，则进行该case处理语句
    default:
        // 如果上面都没有成功，则进入default处理流程
}

```
```
func fibonacci(c, quit chan int) {
	x, y := 1, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)

	go func() {
		for i := 0; i < 6; i++ {
			fmt.Println(<-c)
		}

		quit <- 0
	}()

	fibonacci(c, quit)
}
```
###### 超时
有时候会出现goroutine阻塞的情况，那么我们如何避免整个程序进入阻塞的情况呢？我们可以利用select来设置超时，通过如下的方式实现：
```
func main() {
	c := make(chan int)
	o := make(chan bool)

	go func() {
		for {
			select {
			case v := <-c:
				fmt.Println(v)
			case <-time.After(time.Second * 5):
				fmt.Println("timeout")
				o <- true
				break
			}
		}
	}()

	c <- 666
	//5s
	res := <-o
	fmt.Println(res)
/*
666
timeout
true
*/
}
```
##### Socket编程
###### 什么是Socket
Socket起源于Unix，而Unix基本哲学之一就是“一切皆文件”，都可以用“打开open –> 读写write/read –> 关闭close”模式来操作。Socket就是该模式的一个实现，网络的Socket数据传输是一种特殊的I/O，Socket也是一种文件描述符。Socket也具有一个类似于打开文件的函数调用：Socket()，该函数返回一个整型的Socket描述符，随后的连接建立、数据传输等操作都是通过该Socket实现的。

常用的Socket类型有两种：流式Socket（SOCK_STREAM）和数据报式Socket（SOCK_DGRAM）。流式是一种面向连接的Socket，针对于面向连接的TCP服务应用；数据报式Socket是一种无连接的Socket，对应于无连接的UDP服务应用。
###### 服务器代码
```
import (
	"fmt"
	"log"
	"net"
	"strings"
)

func dealConn(conn net.Conn){
 	defer conn.Close()

 	ipAddr := conn.RemoteAddr().String()
 	fmt.Println(ipAddr,"连接成功")

 	buf := make([]byte,1024)

 	for{
 		n,err := conn.Read(buf)
 		if err != nil{
 			fmt.Println(err)
			return
		}

 		result := buf[:n]
 		fmt.Printf("接收到数据来自[%s]=>[%d]:%s\n",ipAddr,n,string(result))

 		if "exit" == string(result){
 			fmt.Println(ipAddr,"退出连接")
			return
		}

 		conn.Write([]byte(strings.ToUpper(string(result))))
	}
}

func main() {
	listenner,err := net.Listen("tcp","127.0.0.1:8000")
	if err != nil{
		log.Fatal(err)
	}

	defer listenner.Close()

	for{
		conn,err := listenner.Accept()
		if err != nil{
			log.Println(err)
			continue
		}

		go dealConn(conn)
	}
}
```
###### 客户端
```
import (
	"fmt"
	"log"
	"net"
)

func main(){
	conn,err := net.Dial("tcp","127.0.0.1:8000")
	if err != nil{
		log.Fatal(err)
		return
	}

	defer conn.Close()

	buf := make([]byte,1024)
	for{
		fmt.Printf("请输入发送的内容:")
		fmt.Scan(&buf)
		fmt.Printf("发送内容:%s\n",string(buf))

		conn.Write(buf)

		n,err := conn.Read(buf)
		if err != nil{
			fmt.Println(err)
			return
		}

		result := buf[:n]
		fmt.Printf("接收到的数据[%d]:%s\n",n,string(result))
	}
}
```
##### HTTP编程
###### Web工作方式
对于普通的上网过程，系统其实是这样做的：浏览器本身是一个客户端，当你输入URL的时候，首先浏览器会去请求DNS服务器，通过DNS获取相应的域名对应的IP，然后通过IP地址找到IP对应的服务器后，要求建立TCP连接，等浏览器发送完HTTP Request（请求）包后，服务器接收到请求包之后才开始处理请求包，服务器调用自身服务，返回HTTP Response（响应）包；客户端收到来自服务器的响应后开始渲染这个Response包里的主体（body），等收到全部的内容随后断开与该服务器之间的TCP连接
。

Web服务器的工作原理可以简单地归纳为：
- 客户机通过TCP/IP协议建立到服务器的TCP连接
- 客户端向服务器发送HTTP协议请求包，请求服务器里的资源文档
- 服务器向客户机发送HTTP协议应答包，如果请求的资源包含有动态语言的内容，那么服务器会调用动态语言的解释引擎负责处理“动态内容”，并将处理得到的数据返回给客户端
- 客户机与服务器断开。由客户端解释HTML文档，在客户端屏幕上渲染图形结果

######  HTTP协议
超文本传输协议(HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议，它详细规定了浏览器和万维网服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候，就成了我们常说的HTTPS。

###### 地址（URL）
URL全称为Unique Resource Location，用来表示网络资源，可以理解为网络文件路径。
###### 服务器代码
```
import (
	"fmt"
	"net/http"
)

func myHandler(w http.ResponseWriter,r *http.Request)  {
	fmt.Println(r.RemoteAddr, "连接成功")  //r.RemoteAddr远程网络地址
	fmt.Println("method = ", r.Method) //请求方法
	fmt.Println("url = ", r.URL.Path)
	fmt.Println("header = ", r.Header)
	fmt.Println("body = ", r.Body)

	w.Write([]byte("hello go"))
}

func main() {
	http.HandleFunc("/go",myHandler)

	http.ListenAndServe("127.0.0.1:8003",nil)
}
```
###### 客户端
```
import (
	"fmt"
	"io"
	"net/http"
)

func main()  {
	resp,err := http.Get("http://127.0.0.1:8003/go")
	if err != nil{
		fmt.Println(err)
		return
	}

	defer resp.Body.Close()

	fmt.Println("header = ", resp.Header)
	fmt.Printf("resp status %s\nstatusCode %d\n", resp.Status, resp.StatusCode)
	fmt.Printf("body type = %T\n", resp.Body)

	buf := make([]byte,4096)
	var tmp string

	for{
		n,err := resp.Body.Read(buf)
		if err != nil && err != io.EOF{
			fmt.Println(err)
			return
		}

		if n == 0{
			fmt.Println("读取内容结束")
			break
		}

		tmp += string(buf[:n])
	}

	fmt.Println("buf = ",string(tmp))
}
```

##### sync.WaitGroup
WaitGroup 对象内部有一个计数器，最初从0开始，它有三个方法：Add(), Done(), Wait() 用来控制计数器的数量。Add(n) 把计数器设置为n ，Done() 每次把计数器-1 ，wait() 会阻塞代码的运行，直到计数器地值减为0。
```
func main(){
    for i := 0; i < 100 ; i++{
        go fmt.Println(i)
    }
    time.Sleep(time.Second)
}

func main() {
    c := make(chan bool, 100)
    for i := 0; i < 100; i++ {
        go func(i int) {
            fmt.Println(i)
            c <- true
        }(i)
    }

    for i := 0; i < 100; i++ {
        <-c
    }
}

func main() {
    wg := sync.WaitGroup{}
    wg.Add(100)
    for i := 0; i < 100; i++ {
        go func(i int) {
            fmt.Println(i)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```
##### sync.Mutex互斥锁
- Mutex 为互斥锁，Lock() 加锁，Unlock() 解锁
- 在一个 goroutine 获得 Mutex 后，其他 goroutine 只能等到这个 goroutine 释放该 Mutex
- 使用 Lock() 加锁后，不能再继续对其加锁，直到利用 Unlock() 解锁后才能再加锁
- 在 Lock() 之前使用 Unlock() 会导致 panic 异常
- 已经锁定的 Mutex 并不与特定的 goroutine 相关联，这样可以利用一个 goroutine 对其加锁，再利用其他 goroutine 对其解锁
- 在同一个 goroutine 中的 Mutex 解锁之前再次进行加锁，会导致死锁
- 适用于读写不确定，并且只有一个读或者写的场景
##### sync.RWMutex读写锁
- RWMutex 是单写多读锁，该锁可以加多个读锁或者一个写锁
- 读锁占用的情况下会阻止写，不会阻止读，多个 goroutine 可以同时获取读锁
- 写锁会阻止其他 goroutine（无论读和写）进来，整个锁由该 goroutine 独占
- 适用于读多写少的场景

Lock() 和 Unlock()

- Lock() 加写锁，Unlock() 解写锁
- 如果在加写锁之前已经有其他的读锁和写锁，则 Lock() 会阻塞直到该锁可用，为确保该锁可用，已经阻塞的 Lock() 调用会从获得的锁中排除新的读取器，即写锁权限高于读锁，有写锁时优先进行写锁定
- 在 Lock() 之前使用 Unlock() 会导致 panic 异常
- 写锁会阻止其他gorotine不论读或者写进来，整个锁由写锁goroutine占用

RLock() 和 RUnlock()

- RLock() 加读锁，RUnlock() 解读锁
- RLock() 加读锁时，如果存在写锁，则无法加读锁；当只有读锁或者没有锁时，可以加读锁，读锁可以加载多个
- RUnlock() 解读锁，RUnlock() 撤销单次 RLock() 调用，对于其他同时存在的读锁则没有效果
- 在没有读锁的情况下调用 RUnlock() 会导致 panic 错误
- RUnlock() 的个数不得多余 RLock()，否则会导致 panic 错误
- 读锁占用的情况会阻止写，不会阻止读，多个goroutine可以同时获取读锁

##### 交叉编译
GnuWin32


##### 参考文档
- [6个简例带你玩转Golang指针](https://baijiahao.baidu.com/s?id=1598627292221915297&wfr=spider&for=pc)
- [Golang new和 make的区别](https://www.imooc.com/article/46416)
- [Golang 新手可能会踩的 50 个坑](https://wuyin.io/2018/03/07/50-shades-of-golang-traps-gotchas-mistakes/)
- [Golang sync.WaitGroup的用法](https://studygolang.com/articles/12972)
- [标准库文档 —— sync.Mutex](https://studygolang.com/static/pkgdoc/pkg/sync.htm#Mutex)
- [Go基础系列：互斥锁Mutex和读写锁RWMutex用法详述](https://www.cnblogs.com/f-ck-need-u/p/9998729.html)
