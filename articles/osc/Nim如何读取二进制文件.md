```nim
import streams

type
  Alternating = seq[(float, int)]

proc store(fn: string, data: Alternating) =
  var s = newFileStream(fn, fmWrite)
  s.write(data.len)
  for x in data:
    s.write(x[0])
    s.write(x[1])
  s.close()

proc load(fn: string): Alternating =
  var s = newFileStream(fn, fmRead)
  let size = s.readInt64() # actually, let's not use it to demonstrate s.atEnd
  result = newSeq[(float, int)]()
  while not s.atEnd:
    let element = (s.readFloat64.float, s.readInt64.int)
    result.add(element)
  s.close()


let data = @[(1.0, 1), (2.0, 2)]

store("tmp.dat", data)
let dataLoaded = load("tmp.dat")

echo dataLoaded
```
参考资料:[writing/reading binary file in Nim](http://stackoverflow.com/questions/33107332/writing-reading-binary-file-in-nim)
