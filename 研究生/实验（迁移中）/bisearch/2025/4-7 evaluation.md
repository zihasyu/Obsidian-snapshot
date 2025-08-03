

## 0. Compare with the baselines
### Overall compression ratio(baseline ours)
Table:
SA+odess SA+Meta SA+Meta+Odess
### Reduction size breakdown(meta file \* dedup local delta)
Table
### performance
多个折线图
### index overhead
柱子图或者table

## 1. For SA chunking
### 1.1 range Meta chunk size
- [x] 不同Metadata chunk size粒度下的
OCR 和 avgChunkSize （甚至还可以加overhead和performance）
~~chunkNum breakdown~~
-H参数 server3
## 2. For Metadata-guided
### 2.1 Is Metadata-guided
- [x] 是否打开Metadata-guided分别跑一轮perverion的storage变化，来证明该寻找相似块方法是有效的
-t 参数 node2
## 3. For False filter

node 3 -a -b参数
### 3.1 Is false filter
- [x] 3.1证明需要一个false filter，接受过多的delta反而会降低最后的storage//还可以考虑加DCC
完全舍弃false filter后（即全部接受）的per version 的 storage变化
再加一个有false filter的线
y=OCR
x=version
最后是两条折线图
### 3.2 range false filter
- [x] 3.2证明本文设计的false filter能给大部分数据集做一个还ok的过滤机制
false filter是基于统计数据的过滤，这里改为固定值的过滤，给出一个变化范围。并把false filter这个动态变化的方案作一条虚线插进来



## 修改意见
- [x] 2.1处 结尾加个传统方法的workflow
Local compression. Local compression applies lossess com-
pression algorithms, such as LZ4 [11], to compress base
chunks. Delta chunks that are already encoded are not further
compressed, since local compression only achieves marginal
storage savings in this case [12]

- [ ] 2.45处 mtar写详细一点他的动机和怎么做的。（写了写还是感觉没啥好写的）
To address this, the authors proposed mtar (Migratory tar),
a post-processing transformation that separates metadata from
data blocks. By relocating all header blocks to the end of the
file, mtar isolates metadata volatility, enabling content-defined
chunking (CDC) algorithms to deduplicate data blocks more
effectively. Evaluations on real-world datasets (e.g., Linu

- [ ] 2.95处 issue提的有些重复了
These three cases demonstrate a common issue: unique
chunks containing content from the unmodified File 2 cannot
be deduplicated, leading to redundant storage. To quantify this

- [x] 3.1处 table改名，更明确些，此外字号调大
 Percentage of Unique Chunks with Potential Storage Loss

- [x] 5.2处 把取几级path这件事作为一个参数，再
For Meta-chunks: The system extracts up to two di-
rectory levels from one header within each Meta-chunk,
computes a hash value, and records it in the nameHash-
index. For incoming unique Meta-chunks, FineTAR first
attempts to identify matching patterns in the metadata
index before resorting to feature-based methods when
necessary