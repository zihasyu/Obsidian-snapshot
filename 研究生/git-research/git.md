
### https://git-scm.com/book/en/v2 第十章

#### git 对象
blob object只保存了内容没保存文件名
tree object也存了文件名
commit object 存哈希值
#### git 引用
HEAD引用
标签引用
#### git 包文件
你的磁盘上现在有两个几乎完全相同、大小均为 22K 的对象（每个都被压缩到大约 7K）。 如果 Git 只完整保存其中一个，再保存另一个对象与之前版本的差异内容，岂不更好？
事实上 Git 可以那样做。 Git 最初向磁盘中存储对象时所使用的格式被称为“松散（loose）”对象格式。 但是，Git 会时不时地将多个这些对象打包成一个称为“包文件（packfile）”的二进制文件，以节省空间和提高效率。 当版本库中有太多的松散对象，或者你手动执行 `git gc` 命令，或者你向远程服务器执行推送时，Git 都会这样做。 要看到打包过程，你可以手动执行 `git gc` 命令让 Git 对对象进行打包：
https://git-scm.com/docs/pack-heuristics
### https://www.aviator.co/blog/how-git-compresses-files/
#### 压缩一个文件
1. 将文件 v1 分割成小块，并对它们进行哈希处理。您可以构造，其中键是计算出的哈希值，值是具有该哈希码的块的文件偏移量的列表。 `Map<HashCode, List<Offset>>`
2. 从头开始读取文件 v2。取相同大小的块并对其进行哈希处理。通过上一步创建的表，您可以获得文件 v1 可以具有相同块的偏移量。您可以检查这些偏移量，并找到文件此部分的文件 v1 和 v2 之间的最长公共序列。将复制命令输出到 diff 输出。如果找不到公共序列，可以使用写命令，并继续下一个字节。
#### 压缩多个文件
用一些启发式方法找相似文件
- Same-name files are similar
- Files grow over time
- Latest versions are more accessed than older versions
- Keep the deltas shallow
###  https://juejin.cn/post/7272276908379799563
objects文件夹占用了绝大多数空间，存储了Git仓库的对象（commits、trees和blobs）
![[Pasted image 20240512015912.png]]
