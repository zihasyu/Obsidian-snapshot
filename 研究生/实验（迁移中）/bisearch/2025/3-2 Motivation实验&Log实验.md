

### 1. Motivation 实验
from Skip to Accept Ratio=在version1中被忽略且在version2中被接受的断点 / version1中被忽略的断点
Force CutPoint & change Ratio=Vesion2中新出现的强断切点数量 / Version2中的强断的切点总数
Force cutpoint Percentage=Version2中的强断的切点总数/Version2中的总断点个数

| 数据集          | from Skip to Accept Ratio | Force CutPoint & change Ratio | Force cutpoint Percentage |
| ------------ | ------------------------- | ----------------------------- | ------------------------- |
| _automake    | 4.54%                     | 87.50%                        | 2.73%                     |
| _bash        | 29.23%                    | 25.24%                        | 6.16%                     |
| _coreutils   | 23.80%                    | 80.00%                        | 1.48%                     |
| _fdisk       | 23.52%                    | 20.00%                        | 4.13%                     |
| _glibc       | 22.70%                    | 66.66%                        | 2.53%                     |
| _smalltalk   | 18.34%                    | 5.97%                         | 4.75%                     |
| _gcc         | 25.24%                    | 22.77%                        | 7.61%                     |
| _Thunderbird | 24.34%                    | 97.42%                        | 36.17%                    |
| _chromium    | 30.60%                    | 4.07%                         | 10.47%                    |
| _linux       | 23.39%                    | 49.29%                        | 3.52%                     |
| _WEB         | 24.26%                    | 3.17%                         | 3.13%                     |
|              |                           |                               |                           |
### 2. 不同数据集上互补的原因

##### Feature优势区间
同一个version内部有大量不同名但内容极其相似的文件时，Meta方法无法找到。

例子：
WEB内部有大量shtml文件，他们基于提前规定好的几种模板生成，虽名字不同，但有大量冗余代码。
automake，1400个文件中有1280个不同命shell脚本，且这些脚本内容极其相似，应该是批量生成的脚本。

##### Meta优势区间
文件内部有**均匀分布**且易变部分，滚动哈希值易变以至于feature方法会漏掉一些相似度很高的delta块而去做base块。但Meta方法因为识别他们的文件名，可以较好的匹配相似文件。

例子：
Thunderbird Log与Android Log，每一行都会有时间戳。
### 3. 新增Log数据集

![[613469266c2960d2f5118418619d791.png]]