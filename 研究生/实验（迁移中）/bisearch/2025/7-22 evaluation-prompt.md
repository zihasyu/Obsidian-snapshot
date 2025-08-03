
### DCR+DCE

![[Pasted image 20250723091639.png]]

```
\begin{table}[htbp]
    \centering
    \setlength{\tabcolsep}{4pt}
    \small
    \begin{tabular}{|l|c|c|c|c|}
    \hline
    \multirow{2}{*}{\textbf{Datasets}} & \multicolumn{2}{c|}{\textbf{c=5, m=3}} & \multicolumn{2}{c|}{\textbf{c=4, m=5}} \\
    \cline{2-5}
    & \textbf{DCR} & \textbf{DCE} & \textbf{DCR} & \textbf{DCE} \\
    \hline
    Android & 17.71 & 33.22 & 12.37 & 11.87 \\
    \hline
    Automake Tarballs & 25.07 & 90.26 & 23.12 & 43.08 \\
    \hline
    Bash Tarballs & 18.63 & 76.95 & 19.28 & 62.34 \\
    \hline
    Coreutils Tarballs & 26.79 & 87.19 & 19.25 & 34.06 \\
    \hline
    Fdisk Tarballs & 35.93 & 134.58 & 21.52 & 95.11 \\
    \hline
    Glibc Tarballs & 23.36 & 70.73 & 24.25 & 53.92 \\
    \hline
    Smalltalk Tarballs & 26.88 & 97.19 & 25.15 & 84.84 \\
    \hline
    GCC & 23.12 & 88.72 & 21.22 & 35.59 \\
    \hline
    Thunderbird Tar & 16.26 & 17.71 & 14.82 & 16.41 \\
    \hline
    Chromium & 22.03 & 24.87 & 15.83 & 17.11 \\
    \hline
    Linux & 21.41 & 75.92 & 25.24 & 50.82 \\
    \hline
    WEB & 18.42 & 58.37 & 14.99 & 21.74 \\
    \hline
    \end{tabular}
    \caption{DCR and DCE values for BiSearch with different configurations.}
    \label{tab:bisearch_dcr_dce}
\end{table}
```


### breakdown表格原版

```
\begin{table}[htbp]

    \centering

    \setlength{\tabcolsep}{3pt}

    \small

    \begin{tabular}{|l|c|c|c|c|c|c|}

    \hline

    \multirow{2}{*}{\textbf{Datasets}} & \multicolumn{3}{c|}{\textbf{MO}} & \multicolumn{3}{c|}{\textbf{FineTAR}} \\

    \cline{2-7}

    & \textbf{Dedup} & \textbf{Local} & \textbf{Delta} & \textbf{Dedup} & \textbf{Local} & \textbf{Delta} \\

    \hline

    Linux & 7.78 & 11.56 & 55.09 & 6.72 & 7.62 & 84.05 \\

    \hline

    Web & 20.84 & 26.03 & 227.16 & 20.62 & 21.97 & 245.96 \\

    \hline

    Chromium & 15.22 & 35.44 & 112.72 & 14.67 & 15.84 & 128.45 \\

    \hline

    Automake & 2.63 & 3.96 & 18.79 & 2.16 & 2.35 & 23.80 \\

    \hline

    Bash & 2.65 & 3.41 & 9.24 & 1.71 & 1.88 & 10.42 \\

    \hline

    Coreutils & 2.20 & 3.37 & 12.14 & 2.41 & 2.78 & 18.33 \\

    \hline

    Fdisk & 2.14 & 3.26 & 8.72 & 2.18 & 2.65 & 11.61 \\

    \hline

    Glibc & 6.45 & 10.99 & 44.41 & 4.89 & 5.79 & 54.07 \\

    \hline

    Smalltalk & 4.97 & 7.71 & 27.18 & 3.89 & 4.58 & 34.15 \\

    \hline

    GCC & 4.29 & 7.98 & 27.02 & 3.36 & 3.72 & 35.62 \\

    \hline

    \end{tabular}

    \caption{Breakdown of compression ratios by component (MO vs FineTAR)}

    \label{tab:compression_breakdown}

\end{table}

```

### breakdown表格二位小数
```
\begin{table}[htbp]

    \centering

    \setlength{\tabcolsep}{1pt}

    \small

    \begin{tabular}{|l|c|c|c|c|c|c|}

    \hline

    \multirow{2}{*}{\textbf{Datasets}} & \multicolumn{3}{c|}{\textbf{MO}} & \multicolumn{3}{c|}{\textbf{FineTAR}} \\

    \cline{2-7}

    & \textbf{C-Dedup} & \textbf{C-Local} & \textbf{C-Delta} & \textbf{C-Dedup} & \textbf{C-Local} & \textbf{C-Delta} \\

    \hline

    Linux & 7.78 & 1.49 & 4.77 & 6.72 & 1.13 & 11.03 \\

    \hline

    Web & 20.84 & 1.25 & 8.73 & 20.62 & 1.07 & 11.20 \\

    \hline

    Chromium & 15.22 & 2.33 & 3.18 & 14.67 & 1.08 & 8.11 \\

    \hline

    Automake & 2.63 & 1.51 & 4.74 & 2.16 & 1.09 & 10.13 \\

    \hline

    Bash & 2.65 & 1.29 & 2.71 & 1.71 & 1.10 & 5.54 \\

    \hline

    Coreutils & 2.20 & 1.53 & 3.60 & 2.41 & 1.15 & 6.59 \\

    \hline

    Fdisk & 2.14 & 1.52 & 2.67 & 2.18 & 1.22 & 4.38 \\

    \hline

    Glibc & 6.45 & 1.70 & 4.04 & 4.89 & 1.18 & 9.34 \\

    \hline

    Smalltalk & 4.97 & 1.55 & 3.53 & 3.89 & 1.18 & 7.46 \\

    \hline

    GCC & 4.29 & 1.86 & 3.39 & 3.36 & 1.11 & 9.57 \\

    \hline

    \end{tabular}

    \caption{Breakdown of compression contributions by component}

    \label{tab:compression_breakdown}

\end{table}
```

### index compute
% Feature Index:SF(8B)->FP(32B), num = baseChunk*3

% FP Index:FP(32B)->ChunkInfo(88B), num = non-duplicate Chunk

% (only FineTAR) NameHash Index:NameHash(8B)->FP(32B), num = baseChunk




### prompt
```

## 学术论文实验结果段落写作提示词

**角色设定：** 你是一位顶级计算机系统会议(如FAST、OSDI、NSDI)的资深审稿人，擅长写作简洁有力的实验评估段落。

**写作风格要求：**

1. **开头直击要点**：第一句话直接陈述实验目的和关键指标，避免冗长的背景介绍
   - 模板："We compare/evaluate/measure [具体指标] of different approaches."

2. **数据先行，解释后置**：
   - 先客观呈现核心数据结果
   - 再给出简洁的技术原因解释
   - 避免在数据呈现时夹杂过多分析

3. **具体量化比较**：
   - 优先使用倍数表示(如"3.6×", "1.53×")而非百分比
   - 明确指出与哪个baseline的比较
   - 突出最佳和最差情况的具体数值

4. **分类解释模式**：
   - 将结果按数据集特征或系统行为模式分类
   - 每类给出一句话的技术原因
   - 格式："Dataset X achieves Y improvement, since [简洁技术原因]"

5. **异常情况坦诚处理**：
   - 对于性能不佳的情况，直接承认并简要解释
   - 避免过度辩护，体现科学客观性

6. **前瞻性引用**：
   - 结尾指向后续详细分析："In Exp#X (§Y), we study [具体内容] to [具体目的]"
   - 避免在当前段落过度展开细节

7. **长度控制**：
   - 每个实验段落控制在100-150词
   - 删除重复表述和冗余修饰
   - 保持技术术语精确但表达简洁

**禁忌事项：**
- 避免重复描述同一结果
- 不要在系统级实验中归因到具体组件
- 避免过长的技术细节解释
- 不要使用模糊的定性描述

**输出要求：** 
基于以上风格，将给定的实验结果段落重写为符合顶级会议标准的简洁版本，确保技术准确性的同时提高可读性和影响力。

---

这个提示词的核心是**"数据先行，解释简洁，分类处理，前瞻引用"**，体现了顶级会议论文注重效率和清晰度的特点。
```
