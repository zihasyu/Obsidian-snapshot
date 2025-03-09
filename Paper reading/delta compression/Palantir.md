 **Hierarchical Similarity Detection for Post-Deduplication Delta Compression**
#### Idea
1. N-transform或[[Odess]]会将12个ori特征线性变换为3个SF，每个SF hash表由4个原本ori生成而来 所以命中的相似度要求比较高。本文用设置了三层SF hash表。3\*4 4\*3 6\*2，自上而下相似度要求越来越低，但是后面的层能找到原本找不到的小相似块，减少假阴性。//单一的阈值不能同时减少假阳性和假阴性的数量，如果单一的阈值直接用6\*2的话，则本该能找到相似度更高的块时，会命中一些相似度低的块。
2. 本文将SF hash值称作块的metadata，由于比传统方法多两层10（4+6）个表，所以需要减少他的空间开销，于是提出a metadata lifecycle manager会维护下两层的SF的生命周期，定时删除旧SF。这也会导致一些问题，以至于重复的块也要去算SF来查看自己原本的SF也没有被删除。
3. 最后引入了一个假阳性过滤器来丢弃对整体数据减少不利的匹配块，过滤用的指标是自此之前的Compression rabio。
- 在这里的假阳性的定义是不是太苛刻了？原文：flags the base block as a false positive if its compressibility is below the lossless compression ratio averaged over a sliding window of previous chunks.
#### Readme
提出传统方法只有一段SF阈值时，存在在精度与假阴性之间进退两难的问题，然后自然地提出了3层SF hash表的设计作为核心工作。
~~为了解决新增的后两层SF hash表的带来空间开销，他定期删除后两层的hash，这样做是可以做，而且也确实在每种文件的第八个version之后可以把哈希表开销的增长还原回了N-transform的增长程度。但是，实现起来有问题。首先，不可能在chunkinfo里存后两层SF Hash吧，不然本末倒置了，你开始就别在chunkinfo里存不是更好吗？直接少存一倍。那可行的办法就是去找八个版本前的4+6个SF，restore回来然后去重算两轮SF，时间开销太大了。还是说value不只是块号，再加一个版本号，遍历一遍第二层4个表和第三层6个表。~~//我的问题
假阳性的判断问题


#### Limitations of traditional methods：
the first-fit approach of current super-feature-based systems does not allow them to quantify and discriminate the degree of similarity between similar base block candidates for a new block, leaving them stuck in this dilemma.
>目前基于超级特征的系统的第一拟合方法不允许他们量化和区分一个新块的相似的候选基块之间的相似度，这使他们**进退两难**（精度和假阴性）。
>然后本文提出用三层相似度阈值不同的SF Hash表来解决这个问题。
>//很自然的解决方案，但是我觉得应该可以用ori特征去量化相似度，只是不设置阈值很难确定什么时候结束搜索，时间开销太大了，可能这也是为什么NTransform要设置SF吧？逐个比ori太抽象了，还不如这样设三层阈值，但是用ori特征更灵活？这里再想想。
#### The work of this article：
1. Palantir introduces hierarchical super features with different sensitivities to block similarities to find more candidates of similar blocks
![[Pasted image 20240531104902.png]]
2. The overhead of Palantir for storing additional super-features is overcome by exploiting temporal localities of backup streams
>The default setting of Palantir keeps Tier-1 SFs for all versions, Tier-2 SFs for 5 versions, and Tier-3 SFs for 2 versions. Compared to existing solutions with only one SF tier, the additional metadata of Palantir comes from the Tier-2 and Tier-3 SFs of the last five and two versions, respectively. This metadata overhead is a constant and does not increase as more backup versions are stored.
![[Pasted image 20240531105925.png]]
3. Palantir also introduces a false positive filter to discard matching blocks that are counterproductive for the overall data reduction.
>The key idea of Palantir’s false positive filter is to replace the exact comparison with an estimate by comparing the delta compression ratio of the current block (including the lossless compression of the delta encoding library) with that of the previous 𝐿 lossless compression records. If this delta compression ratio is higher than the average value of those 𝐿 records, we mark it as a true positive; otherwise as a false

![[Pasted image 20240531110337.png]]



```c++
void Palantir::CleanIndex()

{

    cout << "clear and the version is " << Version - 5 << " and " << Version - 2 << endl;

  

    // 清理 SFindex2

    for (auto it = SFindex2.begin(); it != SFindex2.end();)

    {

        if (!it->second.empty() && it->second[0].version < Version - 5)

        {

            it->second.erase(it->second.begin());

            SFDelete++;

        }

  

        // 如果 key 对应的 vector 为空，则删除整个 key

        if (it->second.empty())

        {

            it = SFindex2.erase(it);

        }

        else

        {

            ++it;

        }

    }

  

    // 清理 SFindex3

    for (auto it = SFindex3.begin(); it != SFindex3.end();)

    {

        if (!it->second.empty() && it->second[0].version < Version - 2)

        {

            it->second.erase(it->second.begin());

            SFDelete++;

        }

  

        // 如果 key 对应的 vector 为空，则删除整个 key

        if (it->second.empty())

        {

            it = SFindex3.erase(it);

        }

        else

        {

            ++it;

        }

    }

}
```
