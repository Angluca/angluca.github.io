```nim
# plugin.nim

var data = 0

{.push dynlib exportc.}

proc setup*(num:int) =
  data = num
  echo "Plugin Setup!"

proc getData*: int = data

{.pop.}
```

```nim
# app.nim

import dynlib

type
  SetupProc = proc(num:int) {.nimcall.}
  GetDataProc = proc(): int {.nimcall.}

proc testPlugin(path:string) =
  let lib = loadLib(path)
  if lib != nil:
    let setupAddr = lib.symAddr("setup")
    let getDataAddr = lib.symAddr("getData")
    
    if setupAddr != nil and getDataAddr != nil:
      let setup = cast[SetupProc](setupAddr)
      let getData = cast[GetDataProc](getDataAddr)
      
      setup(123)
      echo "Plugin Data: ", getData()
    
    unloadLib(lib)

testPlugin("./libplugin.so") # unix path
testPlugin("./libplugin.dylib")
```

```bash
$ nim c --app:lib plugin
$ nim c -r app

Plugin Setup!
Plugin Data: 123
```
nimcall其实就是fastcall.

如果想使用nim来写dll可以参考:[nim-invocation-example-from-c](
http://nim-lang.org/docs/backends.html#backend-code-calling-nim-nim-invocation-example-from-c)

代码转自 [Nim官方论坛](http://forum.nim-lang.org/t/1400)
