**Speeding up Resemblance Detection for Redundancy Elimination by Fast Content-Defined Sampling**

#### Idea:
把变长分块时用到的CDC方法应用到了特征选择步骤，For example, if we use a condition that ‘fp & 0x1c == 0’, the sampling rate is about 1/8.这样CDS设置采样率这样能避免边界偏移的影响，减少选择特征时的时间开销。设置不同的mask有不同的采样率。
#### Readme
本文的综述部分做的很友好
核心工作其实只有fp&mask用在了特征选择环节。本文嫌弃N-transform太慢，嫌弃finesse是序列不敏感型以至于假阳性高。所以他在N-transform的基础上在“特征选择环节”用了fp&mask的方式筛选留下了1/2^n，其余算法和N-transform本质上相同，所以收获了比N-transform更快，比finesse更准确的效果。
但是为了与N-transform区分开，本文引用了另一篇论文的**gear hash**方法代替N-transform在“hash计算环节”的rabin hash，以及给fp&mask的方式起名为**CDS**。
Odess运转的核心和N-transform相同，只是加了个采样CDS。但确实比N-transform快得多，也没有因此损失压缩率(±2%)。也因为核心还是N转换，其压缩率比finesse要高得多，以及因为采样率的原因，速度还比finesse快。从实验结果而言，Odess确实是比finesse全面优秀的方法；但是看起来没有finesse那么创新与工作量。

#### Limitations of traditional methods：
The computation overhead of N-Transform consists of two parts: calculating the rolling hash across data and applying time-consuming transforms on each hash.
	N-Transform的计算开销包括两部分：计算跨数据的**滚动哈希**和对每个**哈希应用**耗时的转换。
	本文想在时间开销上做突破，具体而言是在计算滚动哈希和选取特征两个步骤减少时间开销。

#### The work of this article：
本提出了一种快速相似度检测方法Odess，它使用**新的内容定义采样方法CDS**来生成一个小得多的代理哈希集，然后在这个**小哈希集**上应用变换（本质上和N-transform一样）。这减少了转换步骤中的计算从成为瓶颈。
![[Pasted image 20240529160718.png]]
同时Odess还利用更快的**Gear散列**来生成滚动散列。因此，Odess在实现高检测精度和高压缩比的同时，大大降低了相似度检测的计算开销。每次滚动rabin hash消耗5+2，gear hash消耗4左右//该方法是reference中第25个论文提出的



#### III. MOTIVATION AND DESIGN
##### motivation
Characterizing and Selecting are very time-consuming for generating fine-grained sequence-sensitive features and there are opportunities to speed up these two steps, while super feature based Indexing is very efficient for looking up similar candidates and there is no need to change it. Therefore, our goal is to reduce calculations in the first two steps.
##### design
- 计算滚动哈希的环节上用gear hash代替rabin hash，但是gear hash是《W. Xia, H. Jiang, D. Feng, and et al., “Ddelta: A deduplication-inspired fast delta compression approach,” Performance Evaluation, vol. 79, pp.258–272, 2014.》提出的。
- 本文真正的**核心工作**是**基于内容采样**(Content-Defined Sampling, CDS),是受到CDC的启发的方法，用于代替选择。For example, if we use a condition that ‘fp & 0x1c == 0’, the sampling rate is about 1/8. 这样做的好处是避免边界偏移所带来的影响，且具有“准确性，一义性”。
**![[Pasted image 20240529154016.png]]**
//后面是综述啥的，当教材用
#### I. INTRODUCTION:
##### a similar framework of obtaining features
- 所有最先进的相似性检测方法都遵循一个类似的框架，即从每个要检测的项目（一个文件或一个块）中获取特征，并基于比较它们的特征来估计两个项目的相似性。这些方法通常可以分为三个步骤：表征、选择和索引。
1. 在特征化步骤中，将收集一个项的一系列内容特征（即所有子块或所有滑动窗口的散列值）。
2. 在选择步骤中，生成的一些特征将作为一个项目的特征进行采样，其中采样技术包括“固定位置、“每个线性变换上的Top-k,Top-1和每个子块上的Top-1。
3. 在索引步骤中，类似的项目将根据它们的特征进行识别。为了简化这个过程，这些特性被组织成索引，例如使用“Super-Feature”、“Bloom Filter”、“Sorting”等技术。
- 在这三个步骤中，前两步侧重于计算和选择项目的特征，最后一步是根据它们的特征寻找相似的增量压缩候选对象。
##### History of traditional methods
1. 相似性检测方法通常为每个项目生成几个特征（即一个块或文件），并根据匹配特征的数量估计项目之间的相似性（程度）。大多数方法生成**sequence-insensitive features**，这意味着两个项目的特征可能是**无序匹配**。例如，块A的第i个特征可以与块B的第j个特征相匹配。对于序列不敏感的特征，估计相似度需要计算每个项的特征集的交集，这使得搜索相似的候选项变慢。
2. **N-Transform**是唯一生成**sequence-sensitive features**的方法，这意味着项目的第i个特征被直接比较。由于序列敏感特性（只考虑位置匹配），N-Transform引入了Super-Feature method,，该方法将特征列表（例如四个特征）的散列作为Super Feature。当检查类似的项目时，Super Feature会直接在项目之间搜索，并且具有任何匹配的Super Feature的项目都是类似的候选项，这是非常简单和快速的。然而，生成序列敏感特征涉及到对从特征步骤生成的所有特征**应用多个线性变换**，选择步骤的计算成本更高。
3. **Finesse** 旨在通过一种近似方法来改进选择步骤的N-Transform，该方法利用了相似项内的细粒度局部性。与N变换相比，Finesse通过**避免在选择步骤中对线性变换**的计算而获得了更高的速度（5×），但它产生了**sequence-insensitive features**。Finesse在索引步骤中应用超级特性（通常需要序列敏感特性），以便更快地搜索类似的候选对象，但Finesse的特征是序列不敏感的。因此，它会导致检测相似候选项的错误（较低的检测效率）和较低的压缩比。
4. **Odess**尝试设计一种**高速生成sequence-sensitive features**的方法，结合上述两种方法的优点。具体来说，我们专注于减少N-Transform的计算，并且一些观察到的性质指导了我们的研究。N-Transform的计算主要包括两部分（表征和选择），并分别对它们进行优化。对于描述步骤，传统的Rabin滚动散列需要对每个窗口更新进行许多计算，我们将其替换为一个更快的滚动散列，称为Gear。对于选择步骤，它的计算与项目的大小有关，我们在选择步骤之前引入了一种新的内容定义抽样方法。通过一致地采样内容定义的哈希值（即Gearhash），然后对代理应用线性变换，从而生成更小的代理（即线性集），从而大大减少了选择步骤的计算。同时，我们验证了与N-Transform相比，我们的内容定义采样率（例如1/128）对检测效率的影响很小


#### II. BACKGROUND AND RELATED WORKS
Detection accuracy reflects the approximation degree between the actual similarity and the estimated similarity. Among these approaches, they share three common steps:
1. **Characterizing**: characteristics of the item (e.g., hash values of all subchunks or all sliding windows) are generated from an item.
2. **Selecting**: an extracting method is applied to reduce the scale of the characteristics, thus features (a.k.a. signature or other terms in some papers) are extracted from characteristics generated from the 1 st step.
3. **Indexing**: item’s features are added to a similarity index.
  ![[Pasted image 20240529104334.png]]
  
  



有个专栏可以看
https://www.bilibili.com/read/cv25913227/