1.  先了解下nim语言: **[Nim官网](https://nim-lang.org/)**, 或 **[Nim中文网](https://nim-cn.com/)**
1.  Nim下载地址: **[Nim官网](https://nim-lang.org/install.html)** 或 **[Nim中文网](https://nim-cn.com/install.html)**
-   下载x32或x64的Nim最新版 和 mingw  
下载完解压后 终端运行nim目录的finish 按y来确认

3. 接着建一个test.nim文件里面写上

```nim
echo "Hello world nim!"
```

4\. 然后打开控制台cd到这个文件夹里运行

```bash
> nim c -d:release test.nim
```

-d:release是可选参数, 不写的话就是默认debug调试模式, 会提示更多内容, 但文件大小也会增加, 如果想要发布程序当然就用release模式

-   可以安装IDE: **[vscode](https://code.visualstudio.com/Download)** 或 [**nimedit**](https://nim-lang.org/nimedit/)
和 **[nimble](https://github.com/nim-lang/nimble)**
工具.  
vscode里搜索安装nim语言插件, 当然也可以用Vim或 其它编辑器  
nimble类似yum的工具, 可以方便安装nim其它工具和库.

5. nimble安装和使用方法:

```bash
> ./koch nimble #先在nim目录运行koch编译安装nimble
...
#安装完后 就可以使用nimble装库了:)

> nimble list #显示所有库列表
> nimble search sdl2 #查找nim的sdl2库是否存在
> nimble install sdl2 #安装nim的sdl2库
> nimble uninstall sdl2 #卸载nim的sdl2库
> nimble path sdl2 #显示sdl2所在的目录
#/home/user/.nimble/pkgs/sdl2
> nimble refresh #可将安装过的包更新最新版本
```

-   **[Nimble 所有包目录](https://nimble.directory/)** 方便查找想要的库和工具

6. 如果你用过C/C++, 可以使用 [**c2nim**](https://github.com/nim-lang/c2nim)把 c/c++库转化成nim库哦

```bash
> nimble install c2nim  # nimble 真方便:)
```

这里还有些nim教程 :)  
[http://my.oschina.net/angluca](http://my.oschina.net/angluca)

### 官方内容
-   [**Nim官网**](http://nim-lang.org)
-   [**Nim 标准库**](http://nim-lang.org/docs/lib.html) (需要基本库函数都在里面了, 像系统文件查找, 容器使用, 环境变量读取等.)
-   [**Nim 教程 (Part I)**](http://nim-lang.org/docs/tut1.html)
-   [**Nim 教程 (Part II)**](http://nim-lang.org/docs/tut2.html)
-   [**Nim 使用手册**](http://nim-lang.org/docs/manual.html)
-   [**Nim 索引大全**](http://nim-lang.org/docs/theindex.html) (nim官方所有内容在_索引大全_和_使用手册_都能找到, 什么忘记了在这两个页面用ctrl+f查找吧.)

### 中文教程

-   [**Nim中文网教程**](https://nim-cn.com/learn.html)_(推荐)_
-   [**liulun的Nim教程翻译1 (站内有更多教程)**](http://www.cnblogs.com/liulun/p/4505563.html)_(推荐)_
-   [**big\_big\_snail的Nim教程翻译1 (站内有更多教程)**](http://blog.csdn.net/dajiadexiaocao/article/details/46340087)_(推荐)_
-   [**tulayang的Nim用户手册翻译**](https://github.com/tulayang/Nim-lang-Manual-CHN)
-   [**Nim类型教程**](https://github.com/tulayang/Nim-lang-Manual-CHN/wiki/Types)_(重要)_
-   [**Nim核心编程**](https://github.com/ScxMes/Core-Nim-programming/blob/master/%E7%9B%AE%E5%BD%95.md)
-   [**Nim手册**](http://101.200.163.149/docs/Nim%20Manual)

### 非常好的E文教程
-  [**Nim basics国外经典入门教程**](https://narimiran.github.io/nim-basics/ "Nim basics 国外经典入门教程")
-   [**Nim by Example**](http://nim-by-example.github.io/)
-   [**Nim和各种语言的一些实用算法和功能**](https://rosettacode.org/wiki/Category:Nim)

### 方便查找nim的库和函数

-   在这里可以用ctrl+f查找库和函数: [**Nim库文档索引**](https://nim-lang.org/docs/theindex.html)
-   QQ群: _[【Nim开发集中营】469329878](https://jq.qq.com/?_wv=1027&k=5Sp38E0)_  有兴趣的朋友可以加入我们一起交流与学习:)
-   **Vim**爱好者 可以装 **[Nim.vim](https://github.com/zah/nim.vim)** 插件 或者 安装 [**AcVim**](https://my.oschina.net/angluca/blog/3169098), 再不用自己花心配置vim泪, 写nim代码都可以简洁提示
