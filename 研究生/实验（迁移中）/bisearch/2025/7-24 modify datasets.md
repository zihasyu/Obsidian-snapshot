### modify datasets

![[Pasted image 20250724103439.png]]
![[Pasted image 20250724104307.png]]

### datasets
Linux C
Web HTML
Automake Shell
GCC C++
coreutils Rust
~~chromium C++~~
~~bash C    //https://github.com/bminor/bash~~
~~Glibc C~~
~~Fdisk C~~
~~Smalltalk C~~

new :
**react JavaScript**
**netty Java**
**Cpython python**
### modify baselines & exp
0.modify datasets
- [x] code
- [x] data
- [ ] paper

1.baseline: mtar(LZ4) , Tar+zip. 去掉fastcdc+任意。
疑问：mtar+LZ4，还是mtar只去重，不做任何种类的压缩？
- [x] code
- [x] data
- [ ] paper

2.restore performance
mtarOnly, mf,mo,FineTAR
- [x] code
- [x] data
- [ ] paper

3.overhead recipe
可以和chunknums放一起讲
- [x] code
- [x] data
- [ ] paper

4.saving那块，再做一个表格，算上recipes大小的overall compression ratio。
- [x] code
- [x] data
- [ ] paper

### data

overall compression ratio

| Dataset | Mtar+Lz4 | MF      | MO      | FineTar         |
| ------- | -------- | ------- | ------- | --------------- |
| react   | 16.8193  | 22.3272 | 24.7041 | 27.6018 (1.12X) |
| netty   | 18.4841  | 35.4586 | 44.5564 | 62.2577 (1.40X) |
| Cpython | 19.53    | 41.1196 | 54.0376 | 61.0846 (1.13X) |


logicalSize / (CompressionSize + Recipe )

| Dataset   | Mtar+Lz4 | MF      | MO      | FineTar       |
| --------- | -------- | ------- | ------- | ------------- |
| react     | 15.768   | 20.5105 | 22.5006 | 26.2775 (1.17X) |
| netty     | 17.3333  | 31.4527 | 38.4092 | 47.6583 (1.24X) |
| Cpython   | 18.1688  | 35.5171 | 44.7592 | 55.3119 (1.24X) |
| automake  | 6.85738  | 13.653  | 17.5027 | 20.6173 (1.18X) |
| coreutils | 5.93123  | 9.37051 | 11.6207 | 16.9004 (1.45X) |
| GCC       | 13.2719  | 20.3047 | 24.6604 | 30.0514 (1.22X) |
| linux     | 19.2252  | 35.5794 | 45.6506 | 68.7539 (1.51X) |


