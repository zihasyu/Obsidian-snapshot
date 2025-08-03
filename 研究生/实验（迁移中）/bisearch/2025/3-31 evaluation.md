Overall compression(baseline ours)

Is false filter

range false filter

Reduction size breakdown(meta file \* dedup local delta)
performance
index overhead

range Meta chunk size

~~chunkNum breakdown~~

SA+odess SA+Meta SA+Meta+Odess

3.3
本质上Metadata-guided Resemblance Detection是我们的方法，feature-based是传统方法。而在SF-chunks Meta-chunks上先用Metadata-guided，nameHash-Index里没有basechunk时，会切换为用feature-based寻找相似块。而CDC-chunks因为是来自一个大文件的多个chunk，直接使用feature-based去寻找相似块。
上述两端原本是基于流程的去描述（先描述Metadata-guided，再讲何时在哪种块上切换到feature-based），但这样表述有些乱了，不如直接从三种类型chunks上分别讲我们是怎么做的。

SF-chunks：在每个SF-chunks前面的Header处获取路径名与文件名，把路径名与文件名求取哈希值，记录在nameHash-index中。unique SF-chunk进入后，会先通过nameHash-index寻找之前是否有相同路径文件名的SF-chunk，有的话用该chunk作为basechunk。没有的话则使用传统的Feature-based方法去寻找。

Meta-chunks：Meta-chunks是16个header块的聚合chunk。取其中一个header的至多次两级路径名求取哈希值，记录在nameHash-index中。unique Meta-chunk进入后，会先通过nameHash-index寻找之前是否有相同路径文件名的Meta-chunk，有的话用该chunk作为basechunk。没有的话则使用传统的Feature-based方法去寻找。

CDC-chunks：因为一个大文件会产生多个CDC-chunks，难以直接通过nameHash-index获取较优的basechunk选择，所以直接使用Feature-based方法去寻找相似块。


### 3.3 before merge
```latex
\subsection{Metadata-guided Resemblance Detection}
Benefiting from the semantic-aware chunking that effectively extracts and preserves metadata, FineTAR introduces a metadata-guided resemblance detection method to enhance delta compression efficiency. This approach leverages the rich metadata information naturally present in tar archives to identify potential similar chunks without relying on computationally expensive content analysis.

During semantic-aware chunking, FineTAR extracts file names (including paths) from the header of each data block, computes their hash values, and stores them in a metadata index. For each incoming SF-chunk, the system attempts to identify chunks with matching names as potential base chunks, leveraging the observation that files sharing identical names typically exhibit high content similarity across backup versions.

This metadata-guided approach offers two key advantages over traditional feature-based methods:

\begin{itemize}
    \item It relies solely on metadata for identification even when content modifications invalidate traditional feature values like content hashes, enabling detection of similarity between files that have undergone changes
    \item It incurs far lower computational overhead than generating rolling hashes and synthesizing Super Features, significantly accelerating the resemblance detection process
\end{itemize}

The approach is particularly effective for common file update patterns in backup scenarios, where modified files often retain their original names and locations. By prioritizing metadata matching, FineTAR can quickly identify similar files from previous backups, enabling efficient delta compression without the computational cost of feature extraction and comparison.

However, not all chunks can benefit from metadata-guided detection. CDC-chunks (from large files) and Meta-chunks inherently lack one-to-one filename mapping, requiring alternative approaches for similarity detection. Additionally, when files undergo renaming or significant structural changes, metadata matching alone may lead to missed opportunities for delta compression. These limitations necessitate a complementary approach that can handle cases where metadata matching is insufficient.

\subsection{Dual-Modal Adaptive Resemblance Detection}
While metadata-guided resemblance detection provides rapid similarity identification, it has limitations that can affect compression efficiency. To address these limitations, FineTAR implements a dual-modal resemblance detection framework that dynamically integrates metadata matching with traditional feature-based methods.

Feature-based detection remains an essential component of FineTAR for several reasons:
\begin{itemize}
    \item When no filename matches exist for an SF-chunk, feature-based detection can identify similar chunks based on content characteristics
    \item CDC-chunks and Meta-chunks lack one-to-one filename mapping, necessitating content-based similarity detection
    \item Files that undergo renaming between backup versions retain similar content but lose metadata continuity
\end{itemize}

The dual-modal framework intelligently switches between the two methods based on context and historical performance. However, metadata-based detection may occasionally produce false positives, leading to suboptimal compression ratios, as metadata similarity does not guarantee content similarity. To address this issue, FineTAR introduces a false detection filter to evaluate the quality of delta-compressed chunks and identify 
```

### 3.3 after merge

```latex
\subsection{Metadata-Feature Dual-Modal Resemblance Detection}

FineTAR employs a dual-modal resemblance detection framework that intelligently combines metadata-based and feature-based approaches to optimize delta compression across different chunk types:

\begin{itemize}
\item \textbf{For SF-chunks:} FineTAR extracts path and filenames from their associated headers and computes hash values for storage in the nameHash-index. When processing a unique SF-chunk, the system first queries this index to identify chunks with matching names from previous backups. This metadata-based approach leverages the observation that files retaining identical names across backup versions typically exhibit high content similarity, enabling efficient delta compression without the computational cost of feature extraction. If no matching names are found, the system falls back to traditional feature-based detection.

livecodeserver

\item \textbf{For Meta-chunks:} The system extracts up to two directory levels from one header within each Meta-chunk, computes a hash value, and records it in the nameHash-index. For incoming unique Meta-chunks, FineTAR first attempts to identify matching patterns in the metadata index before resorting to feature-based methods when necessary.

\item \textbf{For CDC-chunks:} Since large files produce multiple CDC-chunks, maintaining one-to-one filename mapping becomes impractical. Therefore, FineTAR directly applies feature-based detection to identify similar chunks based on content characteristics.
\end{itemize}

This dual-modal framework offers two key advantages over purely feature-based methods. First, it significantly reduces computational overhead by eliminating redundant feature calculations when metadata matching succeeds. Second, it enables effective similarity detection even when content modifications invalidate traditional feature values, particularly useful for files that maintain their names despite undergoing changes.

To address cases where metadata matching produces suboptimal results (as metadata similarity does not guarantee content similarity), FineTAR incorporates a false detection filter that evaluates the quality of delta-compressed chunks. This mechanism dynamically adjusts the detection strategy based on historical performance, ensuring optimal compression efficiency while maintaining high throughput across diverse backup workloads.
```