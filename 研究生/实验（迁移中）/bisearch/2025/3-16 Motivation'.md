
#### File Boundery（周四提到的）

**MTar在Raw Data上仍存在Storage损失**
即便是MTar，在RAW Data上仍然存在这种可能：
①一个chunk同时含有修改文件A与重复文件B，此时该chunk无法去重掉。
②一个chunk完全来自重复文件B，但因为前面修改文件A Chunking时边界有一些偏移，使得重复文件B的开头也有一些chunk无法被去重掉。
动机是原本属于重复文件A的部分，是可以通过调整Chunk边界策略来提升Storage的部分。


设计实验，统计了因这种原因导致无法去重掉的chunk占unique chunk的比例：
这种情况指：一个chunk部分来自重复文件B或完全来自重复文件B

| Dataset     | Percentage | Dataset         | Percentage |
| ----------- | ---------- | --------------- | ---------- |
| linux       | 42.9%      | automake        | 30.9%      |
| WEB         | 53.2%      | bash            | 10.1%      |
| chromium    | 6.46%      | coreutils       | 46.7%      |
| smalltalk   | 36.0%      | fdisk           | 36.5%      |
| gcc         | 26.0%      | glibc           | 34.1%      |
| Android-Log | 0%         | Thunderbird-Log | 0%         |

注：Log数据集没有重复File


#### MTar在RAW data流中仍有FastCDC的边界偏移问题（周一提到的）

都是想指出**MTar在Raw Data上仍存在因边界导致的Storage损失**
但上周一这个比较迂回，是想说明MTar在Raw Data上仍使用Fastcdc，就会因Fastcdc自己的机制遇到Storage与metadata Overhead之间的tradeoff问题。

| Dataset        | Method  | DedupRatio | Overall Compression Ratio | PerChunkSize |
| -------------- | ------- | ---------- | ------------------------- | ------------ |
| linux          | FastCDC | 10.4666    | 64.8147                   | 8722.5176    |
|                | NoSkip  | 11.6823    | 70.0389                   | 5873.8668    |
| WEB            | FastCDC | 26.1049    | 279.2250                  | 8337.3555    |
|                | NoSkip  | 39.5254    | 350.5610                  | 5017.7972    |
| chromium       | FastCDC | 111.3861   | 232.5440                  | 8861.4455    |
|                | NoSkip  | 113.0431   | 228.8360                  | 6058.4545    |
| ThunderbirdTar | FastCDC | 1.0000     | 6.6091                    | 10246.2143   |
|                | NoSkip  | 1.0004     | 5.7310                    | 4874.8427    |
| Android        | FastCDC | 1.0561     | 6.4900                    | 10410.5903   |
|                | NoSkip  | 1.0590     | 5.7088                    | 4430.4031    |
| automake       | FastCDC | 3.2271     | 20.3217                   | 8904.9829    |
|                | NoSkip  | 3.4371     | 21.2461                   | 6236.5959    |
| bash           | FastCDC | 2.7542     | 9.2079                    | 8448.3962    |
|                | NoSkip  | 2.9436     | 9.5010                    | 5809.7255    |
| coreutils      | FastCDC | 2.5098     | 12.3697                   | 9457.6837    |
|                | NoSkip  | 2.6861     | 12.7503                   | 6854.4638    |
| fdisk          | FastCDC | 2.3911     | 8.5506                    | 8413.5411    |
|                | NoSkip  | 2.5548     | 8.6633                    | 5960.9452    |
| glibc          | FastCDC | 8.9632     | 52.7390                   | 8199.1738    |
|                | NoSkip  | 9.8622     | 54.6299                   | 5316.0349    |
| smalltalk      | FastCDC | 6.0695     | 29.3621                   | 8326.7127    |
|                | NoSkip  | 6.7161     | 30.3364                   | 5407.3528    |
| GCC            | FastCDC | 6.0322     | 29.6703                   | 8854.9183    |
|                | NoSkip  | 6.4554     | 30.6071                   | 6094.8926    |


2025 Fast 夏文
2011年 夏文 小文件->大文件 Silo 
看周一的实验怎么不动声色的放进去