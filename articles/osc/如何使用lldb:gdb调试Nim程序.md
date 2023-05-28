* **nim新版本已加入nim-gdb工具方便调试, 具体可以点击**  
[使用nim-gdb调试nim程序](https://my.oschina.net/angluca/blog/3202507l)
  
 
 
一般用nim写程序基本都用echo, repr等命令简单调试, 相信大家也会需要高级调试的时候, 所以在这里介绍如何使用lldb或gdb调试nim程序.

首先输入代码保存test.nim.
```nim
#test.nim
echo "Hello World"
var x = 10
echo "Value of x: ", x
```
然后打开终端.
```bash
$nim c --debugger:native test.nim #用nim编译出带调试内容的程序

$lldb ./test #或者gdb ./test
(lldb)b test.nim:2
(lldb)r #运行后会断到 x = 10 这行
(lldb)n #运行到下一行 echo ...
(lldb)p x_95008 #查看x的值,为什么是x_95008等会再说
$0 = 10
(lldb)p x_95008 = 666 #改变x的值
$1 = 666
(lldb)c
Value of x: 666
(lldb)q #y 退出调试器
```
代码里的变量明明是x为什么变成x_95008了呢? 而且这个数字我是怎么知道的?

  这个疑问首先要了解nim的编译过程是怎样, nim编译文件首先会把nim代码转成c/c++/objc代码放到同目录的nimcache目录里, 然后再交给相关的编译器编译, 所以这个变量真实的名字就是在./nimcache/test.c这个文件里.
```c
//./nimcache/test.c

#line 1 "/Users/xxx/nimcache/test.nim"
	nimln(1, "test.nim");	printf("%s\012", ((NimStringDesc*) &TMP140)? (((NimStringDesc*) &TMP140))->data:"nil");
#line 2 "/Users/xxx/nimcache/test.nim"
	nimln(2, "test.nim");	x_95008 = ((NI) 10);
#line 3 "/Users/xxx/nimcache/test.nim"
	nimln(3, "test.nim");
#line 3 "/Users/xxx/nimcache/test.nim"
	LOC1 = 0;	LOC1 = nimIntToStr(x_95008);	printf("%s%s\012", ((NimStringDesc*) &TMP141)? (((NimStringDesc*) &TMP141))->data:"nil", LOC1? (LOC1)->data:"nil");	popFrame();}
``` 
  这些内容都在test.c文件最下面, line其实就是test.nim文件上的代码行, 相同的x_95008就是那个x的真实变量名了, 更多的操作方法大家可以了解lldb/gdb的使用.

参考资料:
- [Debugging nim](http://hookrace.net/blog/what-makes-nim-practical/#debugging-nim)
- [Debug Nim with GDB](https://internet-of-tomohiro.netlify.com/nim/gdb.en.html)
