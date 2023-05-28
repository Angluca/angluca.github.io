安装
------------
1. 前提有安装nim和mingw等
1.  nim新版本已加入nim-gdb工具方便调试, 没有可以下载放nim/bin下(8.0.0版以上gdb不用nim-gdb貌似也好调试 :)
1. 下载安装python2或3
1.  修改 nim/config/nim.cfg 里 大概214行   gcc.options.debug = "-g3 -O0 -gdwarf-3"  解决调试溢出问题

使用方法
------------

首先我们写个dtest.nim
```
proc foo(x: int): int =
  let y = x + 2
  return y * 10
proc bar(x: int): int =
  if x == 3: return foo(x)
  return x * 100
proc main =
  var a = 1
  let str = "foobar"
  var seq1 = @[0, 1, 2, 3, 4]
  a = bar(1)
  a = bar(2)
  a = bar(3)
main()
```
然后终端运行 调试参数``--debugger:native``编译nim文件  
``nim c --debugger:native dtest.nim``  
编译完使用nim-gdb来调试程序  
``nim-gdb dtest``


使用break或b 来设置断点 ***(输入函数名再按tab键 可以自动完成杂交函数名)***  
![b](https://oscimg.oschina.net/oscnet/up-b14462d526c8cadb948aae21accbaee5241.png "b")  
用run或r 开始运行, 然后会停在第8行断点![](https://oscimg.oschina.net/oscnet/up-9c96929745e141fb229bfa9492ede27a9bb.png)  
list或l 来查看行数周围的代码列表  
![](https://oscimg.oschina.net/oscnet/up-12d62ded6b00f3068b55b9e79c86d172d0c.png)  
next或n 来运行下一行, print或p 查看变量  
![](https://oscimg.oschina.net/oscnet/up-bf30cbf0bba97076fec3925c682e4ed03eb.png)  
step或s 进入函数地址, continue或c运行到断点或结束, finish或fin 运行到跳出当前模块![](https://oscimg.oschina.net/oscnet/up-5a15b6c17545a7f9bd14f9488ff7b5d0ac9.png)  
backtrace或bt 查看模块运行步骤, 从下往上![](https://oscimg.oschina.net/oscnet/up-49d7c4c7a488eb885fbe17db0e832410a79.png)  
info break或i b 和delete或d 查看和删除断点![](https://oscimg.oschina.net/oscnet/up-42e400bc1b206a206266230610e70aec9df.png)  
info locals或 i lo查看当前模块变量![](https://oscimg.oschina.net/oscnet/up-1ac718609f7b60c93fb2d1cb0dd2b6de3b4.png)  
watch或wa 可以让选择查看的变量改变时 变成断点![](https://oscimg.oschina.net/oscnet/up-6cda76553ffdec130c4651ebf5bdbab4e22.png)  
until或u 可以跳出循环模块 像while, for等  
还有更多的命令, 可以详细了解下gdb :)

- 具体参考  
[Debug Nim with GDB](https://internet-of-tomohiro.netlify.com/nim/gdb.en.html "Debug Nim with GDB")
