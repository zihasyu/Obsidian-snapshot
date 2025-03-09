
![[Pasted image 20250307000204.png|400]]

# Software management:
## Motivation
//有个问题我不知道这个Software management在Design2的地方在写，还是在motivation就写。

在Motivation中总结 Software management 场景下有哪些文件{Code,Log,Lib}需要被包含。
然后指出该场景下，数据集有以下特点：
1.大部分文件偏小，文本可读性强。
2.该场景有强烈的备份需求（历史版本code有时候需要回退，过去的Log需要被拿来查看，某些工程依赖于特定版本的链接库）。
3.Code修改都是相对比较小，Log修改较大且均分分布在文件内部。

所以该课题对该场景备份系统从chunking到寻找相似块都做出了特定的改进。
## Design：
Design1:Chunking上，用我们的SAchunking{既有对小文件的File-level，又有对大文件的chunk-level}来代替单纯的Chunk-level，想把之前Tar那套motivation简单搬到这里提一嘴。

Design2:
SAchunking下，我们容易感知到文件的元数据，例如name，我们可以利用其帮助我们寻找相似块，因为我们可以认为name是文件是否相似的强相关因素。
但是Software management中，不同类型的文件又有一些不同的特性，这使得有一些在一些文件中利用Meta方法不如Feature方法有效，反之亦然。
- **Meta优势区间：log文件**
不同日期的同名Log文件，内部有**均匀分布**且易变部分，滚动哈希值易变以至于feature方法会漏掉一些相似度很高的delta块而去做base块。但Meta方法因为识别他们的文件名，可以较好的匹配相似文件。
- **Feature优势区间**：**Code，Lib文件**
同一个version内部有大量不同名但内容极其相似的文件时，Meta方法无法找到。

Design3:（实际上被融入design2了）
这个部分原来写的就可以了，是一个探讨。利用Name寻找相似块虽好，但需要引入淘汰机制(也就是False Filter)，使某个文件更改过大时，有较新版本的base file可供用于delta。



