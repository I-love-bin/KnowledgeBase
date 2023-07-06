# デバッガ検知
## IsDebuggerPresent
- ProcessEnvironmentBlockのBeingDebugFlagをチェックする。
- PEBのオフセット0x2にBeingDebugedが配置されている。
- ```IsDebuggerPresent()```を用いずに、直接PEBを参照する場合もある（```IsDebuggerPresent()```の検出を回避するため）
- ollydbgにおいては、ollyscript```dbh```でBeingDebbugedを0x0に設定することができる。
```C:DetectDbg
  mov    eax, dword [fs:0x30]
  mov    eax, byte ptr [eax+0x2]
```
- [PEB構造体](http://terminus.rewolf.pl/terminus/structures/ntdll/_PEB_combined.html)
## NtGlobalFlag, Heap Flags
### NtGlobalFlag
- fs:\[0x68\]に格納されている。(32bit環境の場合)
- デバッガが使用されていないときは0x0が格納されているが、デバッガが使用されているときは0x70が格納されている。
- NtGlobalFlagの```FLG_HEAP_ENABLE_TAIL_CHECK(0x10)```,```FLG_HEAP_ENABLE_FREE_CHECK(0x20)```及び```FLG_HEAP_VALIDATE_PARAMETERS(0x40)```による。
- ***ntdl!LdrpInitializeExecutionOption()とは？***
- デフォルトのGlobalFlagはgflags.exeや```HKLM\SoftWare\Microsoft\Windows NT\CurrentVersion\Image File Execution Options```から変更可能
```C
  mov  eax, [fs:0x30]
  cmp  dword [eax+0x6b], 0x0
  jne  debugged
```
- [NtGlobalFlagの値と役割](https://learn.microsoft.com/ja-jp/windows-hardware/drivers/debugger/global-flag-reference)
### Heap Flag
- PEB構造体の```ProcessHeap```を参照
- ProcessHeapの実態は、```_HEAP```構造体
- 通常の状態では、```PEB.ProcessHeap.Flags```=0x2かつ```PEB.Process.ForceFlag```=0x0
- ```Flags```及び```ForceFlags```はそれぞれ0xc及び0x10のオフセットに存在する。
```C
  mov eax, [fs:0x30]
  mov ebx, [eax+0x18]
  cmp dword [ebx+0xc],0x2
  jne debugged
```
- [PEB.ProcessHeap構造体](http://terminus.rewolf.pl/terminus/structures/ntdll/_HEAP_combined.html)
### CheckRemoteDebuggerPresent, NtQueryInformationProcess
- プロセスがデバッガにアタッチされているときに、戻り値が非0になる。
```C
BOOL CHeckRemortDebuggerPresent(
  HANDLE hProcess,
  PBOOL  pbDebuggerPresent    ;Pointer to boolean val, if TRUE, debugger is attached.
)
```
# ブレークポイント・パッチ検知
# 解析妨害
# デバッガへの攻撃
# その他のテクニック
# ツールとか
