## Fearture-based  Delta共性

### Delta Compression 系统流程图
![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291513831.png)
Deduplication能过滤掉重复块，即savesize为0，只留下一个recipe大小的存储开销用于日后恢复。所以需要计算SF并去查SFindex表的块都是已经用FP去重过滤后得到的unique chunk。后续会详细介绍独属于对unique chunk的处理，这部分是delta compression system比只去重系统要增加的部分。

### Fearture-based 通用的流程图

![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291506456.png)
每个独特快都要计算SF(若干个ori feature经过线性变换后得到的1个新的feature叫做super feature)，先拿去查表，有相似块的话做delta compression，没有的话把算好的SF插入SFindex中即可，那之后对当前块做local compression后持久化。
### SF-index相关 的通用流程图
![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291526625.png)
再将需要实现的部分总结并抽象出来，发现实际上不同feature-based方法的核心与差异在于SF的计算（Rolling hash，Linear transform）这个方法，此外还要根据SFindex的结构再去实现SF_Find, SF_insert这两个方法。故在实现不同的feature-based方法时重点实现这三个函数，其余很多部分可以复用。

## SF Methods

### N-transform-SF
开山之作，作者写的很随意，拿和同事的对话当引言，真正意义上的master piece，去看原文吧。
实现要点：
- Rolling hash: **Rabin 指纹**去算滚动哈希，对同一个块用12套参数(a,b)求得12个不同的原始特征ori feature
- Linear transform:将12个ori feature分3组，线性变换为3个SF，存储在三个哈希表中。
- 压缩效果较好，时间开销极大
![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291744198.png)

### Finesse
经常被拿来对比的经典作。
实现要点：
- Rolling hash: **Rabin 指纹**去算滚动哈希，将一个chunk分为12个subchunk，那之后用同1套参数对每个subchunk求1个ori feature，这样合计12个ori feature。//只有Finesse做了分子块这个操作，有个隐形的限制是原始块不能小于一定值，这个值是其他方法的12倍大小，对chunking方式有一定限制。
- Linear transform:将12个ori feature先组内排序，再分3组，线性变换为3个SF，存储在三个哈希表中。
-  压缩效果较差，时间开销减少一些
![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291744886.png)

### Odess
在压缩效果和计算开销上都非常务实的作品，但可惜行文中自己的工作量不太够，没能中更好的期刊。
实现要点：
- Rolling hash: **Gear 滚动哈希+CDS抽样**去算滚动哈希，利用CDS使得只用对1/128个基于内容而定的窗口计算哈希值，虽然对同一个块用12套参数(a,b)求得12个不同的原始特征ori feature，但因为是抽样计算，所以耗时比finesse还要短。
- Linear transform:将12个ori feature分3组，线性变换为3个SF，存储在三个哈希表中。
-  压缩效果较好，时间开销减少更多
![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407291747134.png)

### Palantir
提出了一些观点值得思考，但工程角度的总体评价远不如Odess
实现要点：
- Rolling hash: **Gear 滚动哈希+CDS抽样**去算滚动哈希，利用CDS使得只用对1/128个基于内容而定的窗口计算哈希值，虽然对同一个块用12套参数(a,b)求得12个不同的原始特征ori feature，但因为是抽样计算，所以耗时比finesse还要短。
- Linear transform:将12个ori feature分3组，线性变换为3个SF，存储在3个哈希表中。第二层：将12个ori feature分4组，线性变换为4个SF，存储在4个哈希表中。第三层：将12个ori feature分4组，线性变换为6个SF，存储在6个哈希表中。合计13个index。//更多的空间开销和计算开销
- 为减少每个unique chunk有13个SFindex的开销，推出了生命周期机制，第二层的index在插入5个版本后清除，第三层的index在插入2个版本后清除。//更多的计算开销。
- 因清除的存在，palantir提出该设计下需要给重复块也计算SF，来查询是否需要补充第2、3层的SFindex。//更多的计算开销
-  压缩效果短期非常好，长期效果非常差，时间开销极其多。

## Conclusion
大部分改进delta compression的论文都是从改进本章节SF的计算与选择出发的，改进的指标会是吞吐量或者overall compression ratio。
不同的SF组装方法，本质是设计了一个不同相似度阈值，较过低或过高都会有损压缩率。每个块及以其作base的delta chunk的最优阈值是不同的，每个数据集更是不同。当然在小数据集上降低阈值寻找更多的相似块是有利于overall ratio的。
而palantir看似设计了多个阈值，但长远来看还是降低了阈值，在追求optimal overall ratio的道路上这是有害的。

