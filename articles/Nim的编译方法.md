一般的编译参数:  
cc 编译成C语言, c默认为cc.  
cpp 编译成C++语言  
objc 编译成objc语言  
js 编译成javascript脚本, 可以建一个html文件在<script src="test.js"></script>里运行.  
-d:release 进行release编译.  
-r 编译完成后运行程序  
--cincludes:. 包含 当前目录(./)的 c头文件.

具体参考 [cross-compilation](http://nim-lang.org/docs/nimc.html#cross-compilation).

示例:
```bash
> nim c -d:release -r test.nim
> nim objc test.nim
> nim js test.nim
```

伪交叉编译:
伪交叉编译会在nimcache目录里生成跨平台c语言代码, 当前目录生成compile_test.sh编译脚本, 把几个文件都考到要生成的i386:linux机器上运行该sh脚本就会编译成应用程序.  
(其实就是类似于make了一下-.-!)  
示例:
```bash
> nim c --cpu:i386 --os:linux --compile_only --gen_script test.nim
```
交叉编译: (这次不是伪了!)
```bash
> nim c --cpu:arm --os:linux test.nim
```
当然你得设置编译器和链接器这些玩意, 同目录建一个nim.cfg文件里面写上相关参数.
```bash
arm.linux.gcc.path = "/usr/bin"
arm.linux.gcc.exe = "arm-linux-gcc"
arm.linux.gcc.linkerexe = "arm-linux-gcc"
```
这里有一个编译emscripten fc nes模拟器程序例子(具体可看这里[nimnes](http://hookrace.net/nimes/?nes=smb.nes)), 先在nim.cfg设置好参数, 然后运行
```bash
nim -d:release -d:emscripten
```
-d:emscripten其实就是执行@if@end里的内容.
```bash
#nim.cfg
@if emscripten:
  define = SDL_Static
  gc = none
  cc = clang
  clang.exe = "emcc"
  clang.linkerexe = "emcc"
  clang.options.linker = ""
  cpu = "i386"
  out = "nimes.html"
  warning[GcMem] = off
  passC = "-Wno-warn-absolute-paths -Iemscripten -s USE_SDL=2"
  passL = "-O3 -Lemscripten -s USE_SDL=2 --preload-file tetris.nes --preload-file pacman.nes --preload-file smb.nes --preload-file smb3.nes -s TOTAL_MEMORY=16777216"
@end
```
参考资料:[cross-compilation](http://nim-lang.org/docs/nimc.html#cross-compilation)

相关案例:[Porting a NES emulator from Go to Nim](http://hookrace.net/blog/porting-nes-go-nim/)
