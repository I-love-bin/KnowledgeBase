# Shikata-Ga-Nai(Zutto_DEkiru)
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
- 古いFPU命令を使用してアドレスを算出
- 初期化フェーズについては、特別な実行順序はないが、以下の動作絵を行う。
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
### First Deployment
- Algolithm : xor SomeVals
- SomeVals : 今回は初期値に排他的論理和を実行する前の自身の値を加算したのちにデコードを行っている。
```
   0x8048064:	xor    DWORD PTR [eax+0x12],edx
   0x8048067:	add    eax,0x4
   0x804806a:	add    edx,DWORD PTR [eax+0xe]
```
- target address : 0x804806b
- termina addrsee : 0x80481ab
- after first deployment
```
   0x0804806f:	mov    edi,0x3e683237
   0x08048074:	fcmovnb st,st(3)
   0x08048076:	fnstenv [esp-0x354ea70c]
   0x0804807d:	mov    cl,0x4a
   0x0804807f:	xor    DWORD PTR [esi+0x13],edi
   0x08048082:	sub    eax,0xffffffe4
   0x08048085:	add    edi,DWORD PTR [eax+0x38]
   0x08048088:	and    BYTE PTR [ebp-0x45917230],bl
   0x0804808e:	jae    0x8048070
   0x08048090:	movs   DWORD PTR es:[edi],DWORD PTR ds:[esi]
   0x08048091:	cwde   
   0x08048092:	or     BYTE PTR [esi],ah
   0x08048094:	into   
   0x08048095:	inc    esp
   0x08048096:	ret    0x9fef
   0x08048099:	cmp    BYTE PTR ds:0xbdc6ba,dl
   0x0804809f:	pop    ebp
   0x080480a0:	jp     0x80480ce
   0x080480a2:	push   esi
   0x080480a3:	pop    edx
   0x080480a4:	sbb    eax,0x3f0625e6
   0x080480a9:	inc    edx
   0x080480aa:	mov    eax,ds:0xb87d9565
```
## その他のアイディア
```Shikata-Ga-Nai```エンコーダの背景知識
### FPU命令
