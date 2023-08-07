# Shikata-Ga-Nai(Zutto_DEkiru)
## Sample Malware
- malware : shikitega
- MD5 hash : 932df67ea6b8900a30249e311195a58f
- URL : [shikitega](https://bazaar.abuse.ch/sample/e4a58509fea52a4917007b1cd1a87050b0109b50210c5d00e08ece1871af084d/)
## Sample page
- URL1 : [MANDIANT](https://www.mandiant.com/resources/blog/shikata-ga-nai-encoder-still-going-strong)
- URL2 : [Decode Shikata-Ga-Nai with binary ninja](https://medium.com/@acheron2302/writing-binary-ninja-plugin-to-decode-shikata-ga-nai-part-1-df8ceda67fd7)
## Shikata-Ga-Nai Features
- 古いFPU命令を使用してアドレスを算出
- 初期化フェーズについては、特別な実行順序はないが、以下の動作絵を行う。
1. キーを初期化しレジスタに保存
1. FPU命令を使用しEIPを取得
1. デコードする必要があるdwordの数のカウンタを取得
1. （場合によっては開始位置が+4だけずれることもある）
- ```FPU```は
- 排他的論理和を使用してペイロードを復号する。
## Analyze Shikata-Ga-Nai encoded malware
- ```Shikata-Ga-Nai```によてエンコードされたマルウェアの初期動作
```
0x08048054      dac3           fcmovb st(0), st(3)  ; junk instruction
0x08048056      bac374435a     mov edx, 0x5a4374c3  ; junk instruction ?
0x0804805b      d97424f4       fnstenv [esp - 0xc]  ; get eip 
0x0804805f      5e             pop esi              ; 
```
## その他のアイディア
```Shikata-Ga-Nai```エンコーダの背景知識
### FPU命令
