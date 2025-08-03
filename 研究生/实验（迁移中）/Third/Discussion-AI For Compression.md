#### AI For CutPoint
- Storage上FastCDC已经在evaluation中给出了，提升空间不大
- Metadata Overhead方面，本质上是让块变大，但FastCDC 0-4K全跳过这件事本身就是让块偏大的方法，如果我们通过ai决策0-4K中有一些切点不跳过，那么显然Metadata Overhead是变差了的。
- 跳切点本身是FastCDC中Storage与Metadata Overhead的一个tradeoff，Motivate不足。
- Metadata Overhead方面，用**传统方法**扩大块大小可能更好，就这个点很难用ai切入。

#### AI For Delta (Online)
- 未来的数据总量对决策的影响很大，但作为在线系统此时无法预测未来的数据量。
- 因为训练集和测试集的决策偏向不一样，训练出来的模型无法做到通用性。

#### AI For Delta (Offline)
- 需要先证明离线比起在线有较大的storage提升价值，可以尝试拿git做下测试
- 有研究价值，可能可以做，但是没有特别好的方法

#### Related work
**AI+Delta:** DeepSketch: A New Machine Learning-Based Reference Search Technique  for Post-Deduplication Delta Compression

**Metadata overhead：** Dsync: a Lightweight Delta Synchronization  Approach for Cloud Storage Services