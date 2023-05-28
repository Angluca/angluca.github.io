之前有人问如何在nim里使用CreateWindow, 所以在睡觉之前花了些时间写了个winapi的窗口, 方便大家学习使用.  
如果想查找winapi在nim的相关类型或宏可打开windows.nim然后ctrl+f

**(nim0.12版本以后需要用nimble安装oldwinapi才有windows.nim, 位置好象在~/.nimble/pkgs里)**
```nim
import windows

#var hinst = GetModuleHandle(nil)
#echo repr hinst

proc wndfunc(hwnd: HWND, msg: WINUINT, wparam: WPARAM, lparam: LPARAM): LRESULT{.stdcall.} =
  case msg
  of WM_CREATE:
    discard MessageBox(hwnd, "Hello Nim", nil, 0)
  of WM_DESTROY:
    PostQuitMessage(0)
  else:
    result = DefWindowProc(hwnd, msg, wparam, lparam)

var class_name = "app"
var wc:WNDCLASS
wc.style = 0
wc.lpfnWndProc = wndfunc
wc.cbClsExtra = 0
wc.cbWndExtra = 0
wc.hInstance = 0
wc.hIcon = 0
wc.hCursor = 0
wc.hbrBackground = GetStockObject(BLACK_BRUSH)
wc.lpszMenuName = nil
wc.lpszClassName = class_name

if RegisterClass(wc) == 0:
  echo "Failed to register class"

var hwnd = CreateWindow(class_name, "Nim windows", WS_OVERLAPPEDWINDOW, 100, 100, 240, 160, 0, 0, 0, nil)

discard ShowWindow(hwnd, SW_SHOW);
discard UpdateWindow(hwnd)

var msg:MSG
while GetMessage(addr msg, 0, 0, 0) != 0:
  discard TranslateMessage(addr msg)
  discard DispatchMessage(addr msg)
```
