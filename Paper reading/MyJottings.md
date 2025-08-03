#### Week1 5.15-5.19
- 在学BiSearch和finesse，然后跑数据
#### Week2 5.20-5.26
- 改了个多重bisearch，发现确实要saving要更高。此外就是odess作feature效果更好
#### Week3 5.27-6.2
- 读论文，了解了各种feature-base方法
- git是宝藏
#### Week4 6.3-6.10
##### locality-chunking的思考
![[Pasted image 20240607112947.png]]
![[Pasted image 20240607113009.png]]
在更了解chunking过程，以及做了四种增量方法分别用fastCDC与gear分块的实验后。我们观察到相似块的改动可能比我们之前猜测得多得多。块不是文件，基于内容分块也不是真的内容，而仅仅是基于最近13个字节（8KB的块）来决定断点。
- 原本我们预计的是，base=ABC，new=ADB'C，然后利用历史信息的cutpoint来指导切块，好让D独自做base，然后B‘再把B当作base来增量。但实际上，B'的切点也很容易变了，用历史信息指导很容易在base=ABC，new=AB'C的情况找不到B'。
- 考虑到fastCDC的切割方式，也会有这种情况，B'增添了很多内容，新增部分有可以作为断点的内容，于是分裂为B1'B2'，并且B2'因为块大小受到规范化的影响，带走了C的部分内容。此时B1'B2'C'即之后的断点都不再与历史断点相同，无法借助之前的指导。（同时因为边界有所偏移，特征值也会因此错位，乃至B1'B'C'也很难通过finesse或odess找到相似块而作为base，当然长远来看这里作为base可能是更优解？）
- 又或者，重复块容易聚集，改动/新增块也容易聚集，会出现base=ABC，new=ADEFG...，会导致浪费很多时间开销也找不到相同的切点。cutpoint在重复块集群中固然稳定，但在改动块的集群中并不可靠。
- 最重要的一点，最后这项工作肯定要做成一个系统，那么切块一定要批处理，所以借助历史信息切块的操作不能过于“特制化”，目前看来难以实现，并且效果不好。
综上，借助历史信息指导相似块的切割没有良好的效果，但是我们也因此收获了一条以后可能会用到的信息，即**chunking的方式确实对delta compression有影响**。之前的chunking都是针对去重去做去优化，重复块的断点一定会相同。
此外也引发了另一个思考，即**chunk没有太强的metadata可言**。md5码是强信息，但只有重复块用起来才有价值。现有特征提取方式得到的SF与块的CutPoint容易受到边界偏移的影响以至于找不到相似块，也因此是弱信息。chunk不像是文件一样有着路径名+文件名此般的决定性相似查找方式，他的诞生受到chunking方式的影响，特征值也会因重新分块而改变甚多，很难说把所有的优质相似块都找出来。
##### finesse的改进（Successfully but not much improvement）
很容易，之前是分十二子块得到12个ori_feature，把ori_feature分组按大小排序，之后取14710为SF0，25811为SF1，最后4个为SF2，这破坏了序列敏感性。所以不再排序，取1234为SF0，5678为SF1，9101112为SF2。
时间空间开销不变，性能有所提升。
##### superfeature的idea 每个块不固定SF数量，而是每2KB搞一个，然后公用一个散列表⭐idea
既然块内容会不可避免地偏移，块大小也不尽相同，我们为什么要规定每个块的数量要相同呢？为什么SF0SF1SF2要在不同的表里索引呢？我觉得可以按照每2KB内容生成一个SF，然后有几个SF就投入几个到SFindex里，最后一起找。不受偏移和断点影响的SF提取方式。
此外，做了一个测试，将finesse改造为finesse__Adjacency（原本14710为一组，现在1234为一组），改造为序列敏感型版本的finesse，在linux上的结果由11.54→12.61

##### 基于频率分布的攻击（推理明文-密文对）带来的启发
2017 DSN 2022 Infocom这两篇有提到对虚假推理的解决。他也会出现“偏移”的情况，处理方法是调整参数r，来控制去读取某个范围的**明文**。

#### Week5 6.11-6.16

##### 来自git的启发，最优chunking方式，文件为单位⭐⭐⭐idea
考虑看看能不能按照tar编码的方式，给每个文件断开，然后可以用git那边得到的一些启发来做
##### delta compression的改进，为什么只能对一个chunk做delta呢？⭐idea
比如SF表不只命中了一个，SF0命中一位，SF1命中一位，SF2命中另一位，此时可以做个小多层delta而且开销也不会太大
或者load（basechunk+1）一起做
//提出这个是为了解决中间断点的问题，但如果用了tar metadata来切块，不会有这个问题

##### finesse多层效果反而差，odess呢？locality多层效果已知是好的⭐idea(Successful)
![[Pasted image 20240612103400.png]]
是否因为sf方式多层效果一定差呢。

#### Week8 7.1-7.7
##### 控制segment大小
我的想法是类似于fastcdc的上下界控制，expection/2,expection×2分别作为上下界。
①H1D1进入后，size＜下界，则继续前进。
②H3D3进入后，下界< size <上界，则此处为断点。
③H5D5进入前，上界< size，则上一处H4D4为断点。且H5D5单独作为一个segment，一次确定两个segment。
- 此外每次产生两个segment才对，那么expection其实为inputSize的两倍。
- 为了locality需要给headerSegment与dataSegment独立编号
- 独立编号的话，肯定存在某个HeaderSegment对应编号的dataSegment是空，或者反过来（遇到4mb+块的时候），所以两者的块数量需要分别维护。
#### Week9 7.8-7.14
##### 接触到了多线程，还有更metadata的做法。⭐⭐idea
cutpoint用tar，然后headerchunk留着只作单重odess，对data块做log2的多重odess，并且开多线程，因为不会受到plchunk的干扰。(多重比较难绷，可以看下下面的layer设计)

多线程layerdesign单层delta，用tar信息给data文件名做哈希，然后更精准地做delta。这次搞个分层，该文件chunk.deltasize>basechunk.savesize的时候，放弃做delta而是作base。

#### Week10 7.15-7.21
##### 光合基金选择了特征值最宽松的做法，单特征值查找并分组。
在后续大体量数据集上应该效果较差，但在看到数据集前很难说自己用哪个方法。
##### 按字节比例对数据集分类⭐⭐idea
但这个比例该如何链接到分类就不知道了，以及不知道是否真的有这个作用，而且字节比例这个事不知道以前有没有人做过。

#### Week10 7.22-7.28 
##### [[Summary of Chunking]] 写作
此外在出BF的数据与图
#### Week11 7.29-8.4 
##### [[Summary of Feature-based]] 写作

#### Week12 8.5-8. 11
##### 休假
#### Week13 8.12-8.18
##### R语言画图
- 在拿R语言学画图，常见就柱状图和线图。
- 此外右腹气胀去医院了...
- 此外还加了log的Tar数据集，用github记录了构建数据集的方法叫TarDG
#### Week14 8.19-8.25
- 实现了locality method作baseline





#### 11.20
##### 解包多重增量压缩时有没有快捷方法呢
#### 4.29
##### 不解包，直接把压缩后的内容连在一起，作为basechunk去做多层增量压缩。
