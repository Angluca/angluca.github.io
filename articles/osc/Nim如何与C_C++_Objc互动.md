header pragma(头文件指示):
```nim
type
  PFile {.importc: "FILE*", header: "<stdio.h>".} = distinct pointer
  # 导入c语言的 FILE* 指针类型到Nim里用PFile新类型来代替使用.
```
Compile pragma(编译指示):  
直接让nim文件使用c/c++代码文件, 编译的时候会先编译.c文件成.o然后链接让nim也能使用其内容.
```nim
#test.nim
{.compile: "testc.c".}
proc csum(n: cint): cint {.importc.}
echo csum(100)
```
```cpp
//testc.c
int csum(int n) {
    return n + 10;
}
```
Link pragma(连接指示):  
直接链接obj文件.
```nim
{.link: "test.o".}
```
PassC pragma(编译参数指示):  
类似makefile compile flag: -g -Wall
```nim
{.passC: "-Wall -Werror".}
#也可使用外部程序指令
{.passC: gorge("pkg-config --cflags sdl").}
```
PassL pragma(连接参数指示):  
类似makefile link flag: -L/xxx/sdl -lsdl 
```nim
{.passL: "-lSDLmain -lSDL".}
#同样可使用外部程序指令
{.passL: gorge("pkg-config --libs sdl").}
```
Emit pragma(发表指示, 这翻起来还真别扭):
这个指示可以直接在nim代码里运行c/c++/objc等代码, 强大如斯.  
(注释: '"""'3个引号是nim的语法, 在里面的所有内容都是字符串.)
```nim
{.emit: """
static int cvariable = 420;
""".}

{.push stackTrace:off.}
proc embedsC() =
  var nimVar = 89
  # emit里用"`"号(就是~号的按键不要加shift)之间使用nim代码的内容.
  {.emit: """fprintf(stdout, "%d\n", cvariable + (int)`nimVar`);""".}
{.pop.}

embedsC()
```
写类和结构开头要写上/\*TYPESECTION\*/ 或 /\*VARSECTION\*/.
```nim
{.emit: """
/*TYPESECTION*/
struct Vector3 {
public:
  Vector3(): x(5) {}
  Vector3(float x_): x(x_) {}
  float x;
};
""".}

type Vector3 {.importcpp: "Vector3", nodecl} = object
  x: cfloat

proc constructVector3(a: cfloat): Vector3 {.importcpp: "Vector3(@)", nodecl}
```
ImportCpp pragma(C++导入指示):
可用c2nim工具把C/C++库或代码文件转成nim代码.  
和使用importc方法类似, 这些符号会在后面详解, 这里有小教如何在nim里连上鬼火引擎呢.
```nim
# Horrible example of how to interface with a C++ engine ... ;-)

{.link: "/usr/lib/libIrrlicht.so".}

#命名空间都打上, 以后可以不用写这些空间名.
{.emit: """
using namespace irr;
using namespace core;
using namespace scene;
using namespace video;
using namespace io;
using namespace gui;
""".}

const
  irr = "<irrlicht/irrlicht.h>"

type
  IrrlichtDeviceObj {.final, header: irr,
                      importcpp: "IrrlichtDevice".} = object
  IrrlichtDevice = ptr IrrlichtDeviceObj

proc createDevice(): IrrlichtDevice {.
  header: irr, importcpp: "createDevice(@)".}
proc run(device: IrrlichtDevice): bool {.
  header: irr, importcpp: "#.run(@)".}
```
当然也可以不emit命名空间, 可以写成这样.  
(嘛~这些都属于C++的内容了, 想必大家都知道, 但还是多句嘴吧.)
```nim
type
  IrrlichtDeviceObj {.final, header: irr,
                      importcpp: "irr::IrrlichtDevice".} = object
```
Importcpp for procs(c++导入之函数使用):  
单独一个"#"号表示代替第一个或下一个参数.  
"#"号接"."表示使用c++点操作或指针"->"操作.  
"@"号表示代替剩余的参数, 使用时用逗号来分隔.
```nimrod
proc cppMethod(this: CppObj, a, b, c: cint) {.importcpp: "#.CppMethod(@)".}
var x: ptr CppObj
cppMethod(x[], 1, 2, 3)
```
产生的效果等价于
```cpp
x->CppMethod(1, 2, 3)
```
C++的"."或"->"同上面的例子也可以写成:
```cpp
proc cppMethod(this: CppObj, a, b, c: cint) {.importcpp: "CppMethod".}
```
同样也能应用于重载函数
```cpp
//这里应该是代替c++的operator重载操作符+和[]
proc vectorAddition(a, b: Vec3): Vec3 {.importcpp: "# + #".}
proc dictLookup(a: Dict, k: Key): Value {.importcpp: "#[#]".}
```
"'"单引号后面接0~9的数字来替换第i个参数,  第0位则作为返回类型, 可以用来传递c++函数模板.
"'"单引号和数字之间加入"\*"星号表示获得该类型的基类型(所以去掉星号T*就是T类型?), "**"双星表示获得该元素类型的元素类型等. (这句不是很懂, 所以放上原文, E文不错的朋友请告知, 感谢)
  
> An apostrophe ' followed by an integer i in the range 0..9 is replaced by the i'th parameter type. The 0th position is the result type. This can be used to pass types to C++ function templates. Between the ' and the digit an asterisk can be used to get to the base type of the type. (So it "takes away a star" from the type; T* becomes T.) Two stars can be used to get to the element type of the element type etc.
  
```nim
type Input {.importcpp: "System::Input".} = object
proc getSubsystem*[T](): ptr T {.importcpp: "SystemManager::getSubsystem<'*0>()", nodecl.}

let x: ptr Input = getSubsystem[Input]()
```
产生的效果等价于
```cpp
x = SystemManager::getSubsystem<System::Input>()
```
"#@"目前只知道应用于new操作符, 因为new操作符比较特殊吧? 其它功能暂不知.
代替c++的new操作符的示例:
```nim
#'*0#@拆开来就是:'*0为该类型, #@指该操作行为是new吧.
proc cnew*[T](x: T): ptr T {.importcpp: "(new '*0#@)", nodecl.}

# constructor of 'Foo':
proc constructFoo(a, b: cint): Foo {.importcpp: "Foo(@)".}

let x = cnew constructFoo(3, 4)
```
等价于
```cpp
x = new Foo(3, 4)
```
然而, 依赖new的表示式也能用以下方法代替.
```nim
proc newFoo(a, b: cint): ptr Foo {.importcpp: "new Foo(@)".}

let x = newFoo(3, 4)
```
Wrapping constructors(封装构造函数): 
构造/析构函数是c++基础内容, 这里不在阐述, 不懂的话可以翻阅相关资料了解一下.  
加入constructor的pragma即可.
```nim
# a better constructor of 'Foo':
proc constructFoo(a, b: cint): Foo {.importcpp: "Foo(@)", constructor.}
```
Wrapping destructors(封装析构函数):  
```nim
proc destroyFoo(this: var Foo) {.importcpp: "#.~Foo()".}
```
Importcpp for objects(c++导入之对象):  
这里射映了一个C++的map模板类型并声明使用对象
```nim
type
  StdMap {.importcpp: "std::map", header: "<map>".} [K, V] = object
proc `[]=`[K, V](this: var StdMap[K, V]; key: K; val: V) {.
  importcpp: "#[#] = #", header: "<map>".}

var x: StdMap[cint, cdouble]
x[6] = 91.4
```
等价于c++的
```cpp
std::map<int, double> x;
x[6] = 91.4;
```
如果需要更详细的操作, 可以使用"'"加数字来射映(嗯, 那啥上面有解释过).  
```nim
type
  VectorIterator {.importcpp: "std::vector<'0>::iterator".} [T] = object

var x: VectorIterator[cint]
```
等价于
```cpp
std::vector<int>::iterator x;
```
ImportObjC pragma(objc导入指示):  
使用objc需要导入libobjc.a的静态库, passL就是这个作用.    
```nim
# horrible example of how to interface with GNUStep ...

{.passL: "-lobjc".}
{.emit: """
#include <objc/Object.h>
@interface Greeter:Object
{
}

- (void)greet:(long)x y:(long)dummy;
@end

#include <stdio.h>
@implementation Greeter

- (void)greet:(long)x y:(long)dummy
{
  printf("Hello, World!\n");
}
@end

#include <stdlib.h>
""".}

type
  Id {.importc: "id", header: "<objc/Object.h>", final.} = distinct int

proc newGreeter: Id {.importobjc: "Greeter new", nodecl.}
proc greet(self: Id, x, y: int) {.importobjc: "greet", nodecl.}
proc free(self: Id) {.importobjc: "free", nodecl.}

var g = newGreeter()
g.greet(12, 34)
g.free()
```

这里教你如何使用c2nim把c头文件转成nim:[Nim Wrapping C](http://goran.krampe.se/2014/10/16/nim-wrapping-c/)


参考资料:[importcpp-pragma](http://nim-lang.org/docs/nimc.html#additional-features-nodecl-pragma)
