#### Pragmas(编译指示)
编译指示"{."为开始, ".}"为结束, ","号为分隔符, 例如{.cdecl, importc.}

##### deprecated pragma (弃用(分解?)指示)
deprecated指示用来标记为弃用.
```nim
proc p() {.deprecated.}
var x {.deprecated.}: char
```
也可以在声明时使用, 需要定义一个重命名列表.
```nim
type
  File = object
  Stream = ref object
{.deprecated: [TFile: File, PStream: Stream].}
```
##### noSideEffect pragma (无副作用指示)
noSideEffect指示用于标记proc(函数)/iterator(迭代器)为无副作用, 好象是在该函数里使用影响效率的函数或者修改某些变量内容就会出错, 如echo  
未来的发展方向: func可能成为无副作用函数的关键字或语法糖(就是说以后更新正式版可能会加入func这个关键字来声明无副作用函数.)
```nim
func `+` (x, y: int): int
```
##### procvar pragma(过程变量指示)
procvar指示用于标记函数, 让它可以被传递给一个过程(函数)变量.

##### compileTime pragma(编译时指示)
compileTime指示标记的函数和变量仅能在编译时使用, 不会生成代码, 辅助宏来使用.

##### noReturn pragma(无返回值指示)
noReturn标记函数没有返回值.

##### discardable pragma(丢弃返回值指示)
使用discardable标记的函数可以不写返回值或discard.

##### noInit pragma(无初始化指示)
使用该指示的变量不会被初始化.

##### requiresInit pragma(显式初始化指示)
使用该指示的变量需显式的初始化.
```nim
type
    MyObject = object {.requiresInit.}

proc p() =
    # the following is valid:
    var x: MyObject
    if someCondition():
        x = a()
    else:
        x = b()
    use x
```

##### acyclic pragma(非周期指示)
acyclic指示用来标记那些看起来周期循环(其实就是递归使用自身的类型, 像链表什么的)的object(类似c++类)类型为非周期类型, 主要是为了优化效率不让GC(垃圾处理)把这种类型的对象当循环周期部分来处理.
```nim
type
  Node = ref NodeObj
  NodeObj {.acyclic, final.} = object
    left, right: Node
    data: string
```
这个例子把node类型定义为一个树结构, 注意像这种定义为递归的类型 GC会假设这种类型会形成一个周期图, 使用acyclic指示的话 GC将会无视它, 如果你把acyclic指标用于真正的循环周期类型, GC将造成内存泄漏, 当然不会有其它更糟糕的情况(我觉得内存泄漏就很糟糕拉:).  
未来的发展方向: acyclic将成为ref类型属性.
```nim
type
  Node = acyclic ref NodeObj
  NodeObj = object
    left, right: Node
    data: string
```

##### final pragma(最终指示)
类似于c++11的final关键字功能, 其实就是说这个类型不能被继承.

##### shallow pragma(浅拷贝指示)
浅拷贝指示影响类型的语义让编译器允许浅拷贝, 这可能会导致严重的语义问题打破内存安全, 然而, 它可以非常快速的完成作业, 因为nim的语义要求深拷贝序列(seq)和字符串(string), 这是非常费时的, 特别是用序列来建立一个树结构.
```nim
type
  NodeKind = enum nkLeaf, nkInner
  Node {.final, shallow.} = object
    case kind: NodeKind
    of nkLeaf:
      strVal: string
    of nkInner:
      children: seq[Node]
```

##### pure pragma(抽象指示)
对象(object)类型可标记抽象指示, 以便该类型字段在运行时省略类型识别, 主要是为了二进制兼容其它编译语言.  
也可以标记一个枚举(enum)类型, 之后必须写完整才能访问其字段.
```nim
type
  MyEnum {.pure.} = enum
    valueA, valueB, valueC, valueD

echo valueA # 错误, 必须类型字段写完整.
echo MyEnum.valueA # 正确
```

##### asmNoStackFrame pragma(无堆栈帧汇编指示)
函数(proc)可标记asmNoStackFrame指示告诉编译器该函数不生成堆栈帧, 也没有退出声明 如return返回值, 其生成的函数为C语言的 __declspec(naked)或__attribute__((naked))属性函数(取决于使用的C语言编译器).  
注意: 这个指示只能在该函数里使用汇编语句.

##### error pragma(错误指示)
error指示标记用于使编译器输出一个错误消息的内容, 使其在编译发生错误时不一定会终止.  
error指示也可以用来标注一个符号(如一个迭代器(iterator)或函数(proc)), 如果使用符号则发出一个编译时错误, 对于排除那些有效的重载和类型转换的操作是非常有用的.
```nim
#### 如果使用这个函数就会弹出编译时错误.
proc `==`(x, y: ptr int): bool {.error.}
```

##### fatal pragma(致命指示)
fatal指示标记用于使编译器输出一个错误消息的内容, 相对于error指示, fatal指示放哪就在哪里出错.
```nim
when not defined(objc):
  {.fatal: "Compile this program with the objc command!".}
```


##### warning pragma(警告指示)
warning指示标记用于使编译器输出一个警告消息的内容, 警告后继续编译.

##### hint pragma(提示指示)
hint指示标记用于使编译器输出一个提示消息的内容, 提示后继续编译.

##### line pragma(行指示)
line指示标记可以影响注释语言在堆栈回溯跟踪时的可见信息.
```nim
template myassert*(cond: expr, msg = "") =
  if not cond:
    # change run-time line information of the 'raise' statement:
    {.line: InstantiationInfo().}:
      raise newException(EAssertionFailed, msg)
```
如果行指示想使用参数, 则参数需为一个元组(tuple [filename: string, line: int]), 如果无参, 则须使用system.InstantiationInfo()函数.

##### linearScanEnd pragma(直线扫描到最后?)
linearScanEnd指示用来告诉编译器如何编译case语句, 须在case语句里声明:
```nim
case myInt
of 0:
  echo "most common case"
of 1:
  {.linearScanEnd.}
  echo "second most common case"
of 2: echo "unlikely: use branch table"
else: echo "unlikely too: use branch table for ", myInt
```
个人理解可能是在使用case语言里的某个分支使用了linearScanEnd指示就会让该分支之上所有分支的时间效率为O(1), 具体不是很懂, 请参考原句[linearScanend pragma](http://nim-lang.org/docs/manual.html#pragmas-linearscanend-pragma), 懂的请帮忙解惑留言.

##### computedGoto pragma(跳转计算指示)
computedGoto指示用来告诉编译器在while循环里如何编译case语句, 须在循环内声明:
```nim
type
  MyEnum = enum
    enumA, enumB, enumC, enumD, enumE

proc vm() =
  var instructions: array [0..100, MyEnum]
  instructions[2] = enumC
  instructions[3] = enumD
  instructions[4] = enumA
  instructions[5] = enumD
  instructions[6] = enumC
  instructions[7] = enumA
  instructions[8] = enumB
  
  instructions[12] = enumE
  var pc = 0
  while true:
    {.computedGoto.}
    let instr = instructions[pc]
    case instr
    of enumA:
      echo "yeah A"
    of enumC, enumD:
      echo "yeah CD"
    of enumB:
      echo "yeah B"
    of enumE:
      break
    inc(pc)

vm()
```
如例如示computedGoto非常适合作为解释器来使用, 如果使用的C语言编译器不支持跳转计算扩展功能 该指示将被忽略.


##### immediate pragma(立即指示)
immediate指示用于使模板不参与重载解析, 可以在参数调用前不检查语义, 所以可以接收未声明的标识符.
```nim
### 普通模板
template declareInt(x: expr) =
  var x: int

declareInt(x) # 错误: 未知的标识符: 'x'

###立即指示的模板
template declareInt(x: expr) {.immediate.} =
  var x: int

declareInt(x) # 正确
```

##### compilation option pragmas(编译选项指示)
这些列出的指示可以用来覆盖proc/method/converter的代码生成选项.  
目前实现了下列选项(以后可能添加更多的选项)    

| 编译指示        | 可用值   |  描述  |
| :--------   | :-----:  | :----  |
| checks          | on/off | 代码生成时是否打开运行时检测  |
| boundChecks     | on/off | 代码生成时是否检测数组边界   |
| overflowChecks  | on/off | 代码生成时是否检测下溢和溢出  |
| nilChecks       | on/off | 代码生成时是否检测nil指针  |
| assertions      | on/off | 代码生成时断言是否生效  |
| warnings        | on/off | 是否打开编译器的警告消息  |
| hints           | on/off | 是否打开编译器的提示消息  |
| optimization    | none/speed/size | 优化代码的速度或大小, 或关闭优化功能 |
| patterns        | on/off | 是否打开改写术语的模板和宏?  |
| callconv        | cdecl/stdcall/... | 对所有的函数(proc)和函数类型(proc type)指定默认的调用协议 |
示例:
```nim
{.checks: off, optimization: speed.}
### 编译时不会进行运行时检测(checks)和速度优化(optimization)
```

##### push and pop pragmas(推入/弹出指示)
这两个指示需成对使用, 功能类似于选项指示, 作用是临时覆盖那些设置, 例如:
```nim
{.push checks: off.} # 保存旧的设置
### 编译push/pop区域内的代码时不进行运行时检测.
### speed critical
### ... 一些代码 ...
{.pop.} # 恢复旧的设置
```

##### register pragma(寄存器指示)
寄存器指示只能用于变量, 它会声明一个寄存器变量, 告诉编译器该变量应该使用硬件寄存器来提供更快的访问速度, C语言编译器经常会忽略这个功能, 因为它优化的比你快.  
在高度特定的情况下(例如字节码解释器的消息循环)可能会更有效率, 虽然一般没什么卵用。

##### global pragma(全局指示)
global指示类似于c/c++的static关键字功能, 在函数(proc)里使用变量加上global指示可一直使用此变量, 该变量只会在程序运行时初始化一次.
```nim
proc isHexNumber(s: string): bool =
  var pattern {.global.} = re"[0-9a-fA-F]+"
  result = s.match(pattern)
```
每个函数(proc)里的global变量只相对该函数是唯一的, 其它函数里global变量同名不受影响.

##### deadCodeElim pragma(死代码消除)
deadCodeElim指示只应用于整个模块, 它告诉编译器是否对模块激活死代码消除功能.  
在编译时加上--deadCodeElim:on命令时, 所有模块都带有{.deadCodeElim:on}死码消功能.
```nim
{.deadCodeElim: on.}
```

##### pragma pragma(pp指示)
pp指示可以用来声明用户定义的指示, 这非常有用, 因为模板和宏不会影响指示, 它们不能从模块导入.
```nim
when appType == "lib":
  {.pragma: rtl, exportc, dynlib, cdecl.}
else:
  {.pragma: rtl, importc, dynlib: "client.dll", cdecl.}

proc p*(a, b: int): int {.rtl.} =
  result = a+b
```
在这个例子中一个被命名为rtl的指示可以从任意动态库中导入其中的符号(函数或变量)或为动态库生成导出符号.

##### Disabling certain messages(禁止某些消息)
某些用户可能不喜欢nim生成的一些很长的警告和提示, 可用它来关闭那些警告和提示.
```nim
{.hint[LineTooLong]: off.} # 禁止太长的提示.
```
比挨个设置禁止不同的警告要方便多了.

##### experimental pragma(实验性指示)
让你可以使用那些实验性的语言功能, 这些不稳定的功能可能会在出现在未来的稳定版中也有可能永远删除, 喜欢尝鲜的小白鼠可以一试.
```nim
{.experimental.}

proc useUsing(dest: var string) =
  using dest
  add "foo"
  add "bar"
```


##### Foreign function interface(外部函数接口)
Nim的外部函数接口是非常广泛的, 只有部分未来扩展的后端(如LLVM/Javascript)记录在这里.

##### Importc pragma(C语言导入指示)
该指示表示从外部文件导入符号(函数或变量)
```nim
proc printf(formatstr: cstring) {.header: "<stdio.h>", importc: "printf", varargs.}
```
上面是导入C语言的符号, 如果想导入C++的就是importcpp, objc的就是importobjc.

##### Exportc pragma(C语言导出指示)
与importc的功能相反, 主要是作为库导出符号让人调用.
```nim
###fib.nim
proc fib(a: cint): cint {.exportc.} =
  if a <= 2:
    result = 1
  else:
    result = fib(a - 1) + fib(a - 2)
```
```cpp
//maths.c
###include "fib.h"
###include <stdio.h>

int main(void)
{
  NimMain();
  for (int f = 0; f < 10; f++)
    printf("Fib of %d is %d\n", f, fib(f));
  return 0;
}
```
直接编译要加入nimcache目录
```bash
> nim c --noMain --noLinking --header:fib.h fib.nim
> gcc -o m -Inimcache -Ipath/to/nim/lib nimcache/*.c maths.c
```
也可以编译成静态库给c/c++用, linux下可能要加-ldl
```bash
> nim c --app:staticLib --noMain --header fib.nim
> gcc -o m -Inimcache -Ipath/to/nim/lib libfib.nim.a maths.c
```
也可以编译成javascript给html用.
```html
<html><body>
<script type="text/javascript" src="fib.js"></script>
<script type="text/javascript">
alert("Fib for 9 is " + fib(9));
</script>
</body></html>
```
```bash
nim js -o:fib.js fib.nim
```
详细内容请参考:[backends](http://nim-lang.org/docs/backends.html)

##### Extern pragma(外部指示)
类似importc或exportc, extern指示会影响名字识别, 字符串转给extern可以是格式字符串.
```nim
proc p(s: string) {.extern: "prefix$1".} =
  echo s
```
例中把p设为外部名prefix$1(1参数), 类似于win的函数符号协议.

##### Bycopy pragma(被复制指示)
Bycopy该指示可应用于对象和元组, 通过编译器把类型的值转给函数(proc).

##### Byref pragma(被引用指示)
Byref指示可应用于对象和元组, 通过编译器把该类型作为引用传给函数.

##### Varargs pragma(可变参数指示)
varargs指示可以让使用的函数(proc)最后一个参数转变为可变参数(类似c语言printf的fmt), 这个函数使用的nim字符串会自动转换为cstring.
```nim
proc printf(formatstr: cstring) {.nodecl, varargs.}

printf("hallo %s", "world") # "world" 将转换为cstring
```

##### Union pragma(联合指示)
union看起来像c++的union用法, 可应用于任何对象(object)类型, 共享字段, 暂不支持GC和继承.  
未来发展的方向: 将允许GC检测和控制内存.

##### Packed pragma(封包指示)
packed指示可应用于任何对象(object)类型, 它确保对象的字段头尾相接连续保存在一段内存内, 这适用于网络包传输和硬件驱动, 封包指示目前还不能使用继承和GC管理内存.
未来发展的方向: 将支持继承, 如果封闭的内部有使用gc的话将会在编译时提示错误. 

##### Unchecked pragma(无检测指示)
Unchecked指示可以让该数组不对边界检测, 这个非常有用, 适合在你想要一个灵活大小却又不确定大小的数组时使用.
```nim
type
  ArrayPart{.unchecked.} = array[0..0, int]
  MySeq = object
    len, cap: int
    data: ArrayPart
```
类似C语言的这个代码.
```c
typedef struct {
  NI len;
  NI cap;
  NI data[];
} MySeq;
```
无检测的数组类型没有GC内存管理.  
未来的发展方向: 无检测的数组将支持GC内存管理. 

##### Dynlib pragma for import(动态库的导入指示)
dynlib指示可结合importc指示从动态链接库里导入符号(函数,变量等)(\.dll是win的, \.so是linux/unix的动态库扩展名)
```nim
proc gtk_image_new(): PGtkWidget
  {.cdecl, dynlib: "libgtk-x11-2.0.so", importc.}
```
同样支持版本控制.
```nim
proc Tcl_Eval(interp: pTcl_Interp, script: cstring): int {.cdecl,
  importc, dynlib: "libtcl(|8.5|8.4|8.3).so.(1|0)".}
```
运行时会依照这个循序搜索动态库文件.
```bash
libtcl.so.1
libtcl.so.0
libtcl8.5.so.1
libtcl8.5.so.0
libtcl8.4.so.1
libtcl8.4.so.0
libtcl8.3.so.1
libtcl8.3.so.0
```
动态库导入不仅支持常数字符串, 同样支持一般的字符串表达式.
```nim
import os

proc getDllName: string =
  result = "mylib.dll"
  if existsFile(result): return
  result = "mylib2.dll"
  if existsFile(result): return
  quit("could not load dynamic library")

proc myImport(s: cstring) {.cdecl, importc, dynlib: getDllName().}
```
注意: 像libtcl(|8.5|8.4).so只支持在常数字符串, 因为它是预编译.  
注意: 因为初始化循序的问题, 运行时传递变量给该指示将会失败.  
注意: 动态库导入可以被 --dynlibOverride:name 命令行选项覆盖, 查看用户编译手册了解相关信息.

##### Dynlib pragma for export(动态库的导出指示)
dynlib指示也能能把符号导出给他人使用, 该指示没有参数且必须结合exportc指示使用.
```nim
proc exportme(): int {.cdecl, exportc, dynlib.}
```
动态库导出只在程序通过命令行选项 --app:lib 编译成动态链接库才有用.


#### 参考资料:[nim manual-pragmas](http://nim-lang.org/docs/manual.html#pragmas)

