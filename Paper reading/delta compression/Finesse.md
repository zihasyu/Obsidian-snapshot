**Fine-Grained Feature Locality based Fast Resemblance Detection for Post-Deduplication Delta Compression**

#### Idea:
把chunk12等分为subchunk，然后每个subchunk分别找1个最小值特征，0-2，3-5，6-8，9-11分别取最大值放进SF0，取次大值放进SF1，最小值放进SF2。每个分组的4个特征求hash，得到相应的SF。这样比N-transform的线性变换要节省一些时间。
![[Pasted image 20240612092847.png]]
#### Readme
这样做每个块最小为576，N变化类的方法最小为60，而且这种方法序列不敏感以及效果很差，难以找到足够的相似块。

#### Limitations of traditional methods：
The computation overhead of N-Transform consists of two parts: calculating the rolling hash across data and applying time-consuming transforms on each hash.
	N-Transform的计算开销包括两部分：计算跨数据的**滚动哈希**和对每个**哈希应用**耗时的转换。
	本文想在时间开销上做突破，具体而言是在计算滚动哈希和选取特征两个步骤减少时间开销。


