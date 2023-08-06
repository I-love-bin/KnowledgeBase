# Shikata-Ga-Nai(Zutto_DEkiru)
## Sample Malware
- malware : shikitega
- MD5 hash : 932df67ea6b8900a30249e311195a58f
- URL : [shikitega](https://bazaar.abuse.ch/sample/e4a58509fea52a4917007b1cd1a87050b0109b50210c5d00e08ece1871af084d/)
## Sample page
- URL1 : [MANDIANT](https://www.mandiant.com/resources/blog/shikata-ga-nai-encoder-still-going-strong)
- URL2 : [Decode Shikata-Ga-Nai with binary ninja](https://medium.com/@acheron2302/writing-binary-ninja-plugin-to-decode-shikata-ga-nai-part-1-df8ceda67fd7)
## Shikata-Ga-Nai Features
- 古いFPU命令を使用してアドレスを計算する。
- 排他的論理和を使用してペイロードを復号する。
## Analyze Shikata-Ga-Nai encoded malware
```First steo of shikitega malware
│           0x08048054      ba2aeb4935     mov edx, 0x3549eb2a
│           0x08048059      ddc5           ffree st(5)
│           0x0804805b      d97424f4       fnstenv [esp - 0xc]
│           0x0804805f      5f             pop edi
```
