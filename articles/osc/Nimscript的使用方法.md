类似使用python, lua等动态语言, 方便执行终端命令等.
```nim
#test.nims
#指令 任务名, 描述: 执行内容
task mytest, "test nims":
  switch("verbosity", "2")
  echo "script test"
echo "test"
#exec "nim -v"
```
终端执行
```bash
>nim test.nims
#显示
test
>nim mytest test #nim 任务名 文件名或文件全名
#显示更详细的内容, 包括gc
script test
test
...
```
想了解更多内容请看

[Using-nimscript-for-configuration](https://github.com/nim-lang/Nim/wiki/Using-nimscript-for-configuration)

[nimscript doc](https://nim-lang.org/docs/nimscript.html)
