> ###### 更多nim教程和技巧, 可以关注:)  「[www.angluca.com](http://www.angluca.com)」
###### 函数名的使用

```nim
proc my_func(a:int): int =
  return a + 100

echo m_y_f_UN_C(1)
echo myFunc(2)
echo myfunc(3)
```

上面的代码编译是正确的. 原来函数名是忽略大小写和下划线的, 所以爱怎么用函数名就怎么用吧, 峰驼和下划线党不会起冲突了. (注意: nim里名对首字母大小写敏感, 而且首字不能用下划线.)

###### 使用函数可以不用discard
使用pragma: discardable 的函数可以直接使用了.
```nim
proc test(): int {.discardable.} = return 9

test()
#discard test() #如果没有discardable需这样写.
```

###### 数组的使用 nim的数组比c语言的要强大, 可以不从0开始, 也可以是单字母(虽然都一样:)
```nim
var int_array : array[42..44, int]
int_array[43] = 100
# int_array[45] = 1 #错误 超过定义的范围了 
echo int_array[42] #0
echo int_array[43] #100
int_array = [12,42,66] #同样也可以这样赋值
for i,v in int_array:
  echo i, "=", v #i是索引, v是值
#结果
#42=12
#43=13
#44=14
var ch_array : array['A'..'z', int] #从A-Z-a-z
ch_array['a'] = 111
echo ch_array['a'] #111
```
###### 枚举值的查找 nim可以直接用字符串得到枚举值, 省掉不少麻烦.
```nim
import strutils
type MyEnum = enum enA, enB, enC, enuD, enE
doAssert parseEnum[MyEnum]("enu_D") == enuD #还记得使用函数可名不分下划线和大小写么.
doAssert MyEnum(0) == enA
doAssert enA.ord == 0
doAssert $enA == "enA"
```
###### enum的复杂使用 很多时候可以简化功能的实现方法.
```nim
type
    MyEnum = enum
        valueA = (0, "my value A"),
        valueB = "value B",
        valueC = 2,
        valueD = (3, "abc")

echo ord(valueA)
echo ord(valueB)
echo ord(valueC)
echo ord(valued)

echo (valueA)
echo (valueB)
echo (valueC)
echo (valued)
```

显示结果
```bash
0
1
2
3
my value A
value B
valueC
abc
```

###### 对象的多态性 这是其它语言都有的功能, 大家刚用nim也许不知道怎么用, 所以写在这里方便大家学习使用.
```nim
type
  ObjA = object of RootObj
  ObjB = object of ObjA
  ObjC = object of ObjB

method pt(obj:ObjA) =
  echo "A"
method pt(obj:ObjB) =
  echo "B"
method pt(obj:ObjC) =
  echo "C"

proc testObj(obj:ObjA)=
  obj.pt
var
  a = ObjA()
  b = ObjB()
  c = ObjC()

a.testObj
b.testObj
c.testObj

#结果
#A
#B
#C
```

###### 函数参数的应用 函数形参加var是可修改, 不加就是不可改. (类似c++的引用)
```nim
proc test(a:int, b:var int) = #a为不可改, b为可修改参数
   a += 1 #错误! a是不可改的
   b += 1 #正确
```

###### 模块名的应用 强制用户包含模块名使用其中内容
```nim
from os import nil
echo os.isHidden("~/") #正常
echo isHidden("~/") #错误, 必须加os模块名
```

###### 字符串转数字 字符串转化为各种变量, 还有更多的转化变量函数可以去strutils.nim里查看, 而数字转字符串只需变量前加$.
```nim
import strutils
var num = parseInt("123")
var fnum = parseFloat("5.23")
echo num
echo fnum

#123
#5.23
```

###### cstring和string相互转换
```nim
var str: string = "Hello!"
var cstr: cstring = str
var newstr: string = $cstr
```
string是可以直接转为cstring使用, 而cstring转string则需要加上$符号.

###### 巧用seq转cstring 有时候想把数组当字符串用? 教你简单又粗暴的方法.
```nim
import typetraits
var sequ = @['h', 'e', 'l', 'l', 'o']
sequ.add('!')

echo sequ.type.name
echo sequ

let str = cast[cstring](addr(sequ[0]))
echo str.type.name
echo str

#seq[char]
#@[h, e, l, l, o, !]
#cstring
#hello!
```

###### 对象类型的判断
```nim
import typetraits
type
    Person = object of RootObj
        name*: string   # * 表示能在其它模块里使用, 公有
        age: int        # 只能本文件用, 私有
    Student = ref object of Person  
        id: int                       
var
    student: Student
    person: Person
assert(student of Student)  # of 值为 true, is 也可以
assert(student of Person)   # of 可以判断继承类型所以为 true, is 则类型必须相同所以为false.
echo student.type.name # 名为"Student"

var str = "test"
assert(str is string) #值为 true
```

of可以判断是否为继承类, is只能判断是否为当前类型.

###### 手动释放非追踪对象 特别注意: 如果一个非追踪对象包含了追踪对象(比如追踪引用、字符串或者序列),为了正确地释放每一块内存,在手动释放非追踪内存前必须调用 GC_unref()
```nim
type
    Data = tuple[x, y: int, s: string]
# 为 Data 在堆上分配内存
var d = cast[ptr Data](alloc0(sizeof(Data)))
# 通过 GC 在堆上创建一个新的字符串
d.s = "abc"
# 告诉 GC 该字符串不再需要
GC_unref(d.s)
# 释放内存
dealloc(d)
```

如果在 dealloc(d) 之前没有调用 GC_unref(d.s), 那么这个字符串内存永远不会释放. 上面的例子也给出了底层编程的两个重要特点: sizeof() 函数返回一个类型的字节大小, cast\[\] 可以使编译器强制把内存用某个特定类型表示, 只有必要的情况下使用 cast\[\], 它破坏类型安全且容易滋生 bug.

注意: 上面的例子可以工作是因为其内存已经通过 alloc0() 初始化为 0, 也因此 d.s 被初始化为 nil 并且可以对其赋值.

###### 禁止类型对象使用nil值 所有的值可以是 nil 的类型, 可以通过 not nil 声明为禁止使用 nil 值.
```nim
type
  PObject = ref TObj not nil
  TProc = (proc (x, y: int)) not nil

proc p(x: PObject) =
  echo "not nil"

p(nil) # 这里提示异常

var x: PObject
p(x) # 这里也是

var str:string not nil = nil # 这还是
```

###### 内存区域划分 ref 类型和 ptr 类型可以获得一个可选的区域标注, 使用区域标注必须是个对象类型.

在开发操作系统内核时,划分用户空间和内核空间是非常有用的.
```nim
type
    Kernel = object
    Userspace = object
var a: Kernel ptr Stat
var b: Userspace ptr Stat
# 下面的内容不能编译,因为指针类型不兼容.
a = b
```
如例子所示,ptr 也能用做二元操作符,region ptr T 是 ptr\[region, T\] 的简写.  
为了简化泛型编码,ptr T 表示 ptr\[R, T\] (任意 R) 的派生子类.  
作为特殊的类型规则,ptr\[R, T\] 不兼容 pointer.
```nim
# from system
proc dealloc(p: pointer)
# wrap some scripting language
type
    PythonsHeap = object
    PyObjectHeader = object
        rc: int
        typ: pointer
    PyObject = ptr[PythonsHeap, PyObjectHeader]

proc createPyObject(): PyObject {.importc: "...".}
proc destroyPyObject(x: PyObject) {.importc: "...".}

var foo = createPyObject()
# type error here, how convenient:
dealloc(foo)
```
未来方向:  
内存区域也可以对 string 和 seq 有效 内置区域比如 private global local 在即将到来的 OpenCL 会更有用 内置区域可以构造 lent 和 unique 指针 赋值操作符可以附加到区域,以生成合适的写屏障.这意味着 GC 可以彻底在用户空间实现.
###### 函数类型的约定 函数类型是一个指向函数地址的指针。允许把 nil 作为一个函数的值。Nim 语言运用函数类型来达成函数式编程语言的技巧。
```nim
proc printItem(x: int) = ...
proc forEach(c: proc (x: int) {.cdecl.}) =...
forEach(printItem)  # 不能编译，因为调用约定不同
```
```nim
type
    OnMouseMove = proc (x, y: int) {.closure.}
proc onMouseMove(mouseX, mouseY: int) =
    # 使用默认的调用约定
    echo "x: ", mouseX, " y: ", mouseY
proc setOnMouseMove(mouseMoveEvent: OnMouseMove) = discard
# ok，onMouseMove 使用默认的调用约定，该约定兼容  {.closure.}
setOnMouseMove(onMouseMove)
```

一个极其微妙的细节是，函数的调用约定会影响类型的兼容：拥有相同调用约定的函数类型才能够彼此兼容。作为一个特殊的扩展，一个使用 {.nimcall.} 调用约定的函数，可以作为参数传递给一个期望调用约定是 {.closure.} 的函数参数。

###### 用type使用对象类型 type 操作符用来获取一个表达式的类型（同 C 语言的 typeof）：
```nim
var x = 0
var y: type(x) # y has type int
```

type 也可以用来确定一个 proc/iterator/converter 调用 c(X) 的结果类型:
```nim
import strutils

# strutils contains both a ``split`` proc and iterator, but since an
# an iterator is the preferred interpretation, `y` has the type ``string``:
var y : type("a b c".split)
```

###### 局部变量当全局变量使用 类似c/c++的static, 生存周期变成全局, 初始化也只会执行一次, 方便测试功能
```nim
proc getInt(): ptr int=
   var inst {.global.} = cast[ptr int](alloc(4))
   inst[] = 100
   result = inst

echo repr getInt()
echo repr getInt()
```

###### 简易使用任意数组和seq当函数参数
```nim
proc test(v: array or seq) = #或者 array|seq
  echo repr v
  return

var sz:array[9999999, int32]
var sq:seq[string]
test(sz)
test(sq)
```
用这种方法可以直接使用array或seq而不用定义变量类型了哦(当然也可以使用auto).
```
type class      matches

object          any object type
tuple           any tuple type
enum            any enumeration
proc            any proc type
ref             any ref type
ptr             any ptr type
var             any var type
distinct        any distinct type
array           any array type
set             any set type
seq             any seq type
auto            any type
```
需要两个以上类型合并才能使用这个方法, 其它类型也可以, 具体还得自己去体验.

###### defer延迟操作 把前面打上defer的语句推迟到当前块最后操作, 多defer越前的越后执行, defer内顺序执行, 可以替代的try,finally.
```nim
var f = open("numbers.txt")
defer: close(f)
f.write "abc"
f.write "def"
```

类似这个效果
```nim
var f = open("numbers.txt")
try:
  f.write "abc"
  f.write "def"
finally:
  close(f)
```

###### for的步数操作 0..10用习惯了偶尔会忘记怎么使用i+=2步数的操作, 可以自己写个iterator, 但语言自带不用白不用.
```nim
for i in countup(0, 10, 2): #降阶就是countdown
   echo i
```

类似c的
```c
for(int i=0; i<=10 i+=2)
   printf("%d\n", i)
```

###### AST辅助宏 dumpTree宏可以用来辅助AST的使用.
```nim
dumpTree:
  echo "en!"
  var x = 100
  write(stdout, "x")
  writeln(stdout, x)
```
```bash
结果:
StmtList
  Command
    Ident !"echo"
    StrLit en!
  VarSection
    IdentDefs
      Ident !"x"
      Empty
      IntLit 100
  Call
    Ident !"write"
    Ident !"stdout"
    StrLit x
  Call
    Ident !"writeln"
    Ident !"stdout"
    Ident !"x"
```

###### from导入模块的同时制定别名 一般制定别名都是用
```nim
import xxx as x
```

使用 from导入模块同样也可以制定别名哦, 这样不用写比较长的模块名了.
```nim
from xxx as x import nil
```

from导入模块的内容可以导入多个, 用逗号分隔.
```nim
#导入xxx里的x,y,z
from xxx import x,y,z
```

###### x64下编译x86程序
```bash
nim c --cpu:i386 -t:-m32 -l:-m32 xxx.nim
```
如果不加后面的两个-m32就会弹错, 也许以后会改的容易些.
###### 快速得到文件,路径,后缀名
```nim
import os
var appname = getAppFilename()

var (dir, name, ext) = splitFile(appname)
echo dir, ", ", name, ", ", ext
# c:\users\xxx, test, .exe

echo appname.extractFilename()
# test.exe
```

###### 把array当指针操作的正确姿势 当想把nim数组初始化时, 会c的朋友肯定直接会把它memset, 但nim里指针就是指针, 得用ptr/ref得到才行. (如真想当c的类型用可用cstring)
```nim
var inputs = cast[ptr array[10,int]](alloc0(sizeof(int) * 10))
inputs[0] = 99
echo repr inputs
#echo cast[cstring](inputs)
dealloc(inputs)

var test = alloc0(10)
zeroMem(test, 10)
echo repr cast[ptr array[10,char]](test)
#echo cast[cstring](test)
dealloc(test)
```
```bash
ref 000000000018F050 --> [99, 0, 0, 0, 0, 0, 0, 0, 0, 0]
ref 0000000000191050 --> ['\0', '\0', '\0', '\0', '\0', '\0', '\0', '\0', '\0', '\0']
```
如果没有ptr, 上面的数组就不是正确的地址, 而是指在错误的区域显示一堆乱数字, 所以不要用c的思想来看nim的类型变量.

