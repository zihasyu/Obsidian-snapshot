
### CDF 累积分布函数
#### python CDF图
```python
import matplotlib.pyplot as plt  
import numpy as np  
  
filename = 'pic1.txt'  
data = np.loadtxt(filename, usecols=1)  # usecols=1 表示读取第二列（索引从0开始）  
  
# data = np.array([2, 3, 7, 6, 5, 0])  
sorted_data = np.sort(data)  
cumulative_prob = np.arange(1, len(sorted_data) + 1, 1)/float(len(sorted_data))  
print(sorted_data)  
print(cumulative_prob)  

#只显示mask范围的数据
mask = cumulative_prob < 0.9  
filtered_cumulative_prob = cumulative_prob[mask]  
filtered_sorted_data = sorted_data[mask]  
  
plt.plot(filtered_cumulative_prob ,filtered_sorted_data)  
plt.xlabel('Improvement ratio')
plt.ylabel('CDF')
plt.savefig('plot.png')
plt.show()

```
#### Shell awk 
```shell
#Base & Delta as ratio 1,we only need to record their num
#NotFound & BadDelta need to be recorded more, such as their num, per chunk oriMethodSize/BFsize,and get sum(oriMethodSize/BFsize)/num
#pic1: per chunk oriMethodSize/BFsize .txt
#pic2: NotFound & BadDelta chunknum/Sumchunknum , NotFound & BadDelta sum(oriMethodSize/BFsize)/num

Lz4Sum=0
BFSum1=0
OdessSum=0
BFSum2=0
NotFoundnum=0
Badnum=0
Basenum=0
Deltanum=0
chunknum=0
# pic1 tag oriMethodSize/BFsize
NoFoundRatioSum=0
BadRatioSum=0
awk -F, '
{
    if ($3 == "NoFound") {
        Lz4Sum += $5
        BFSum1 += $7
        print "NoFound " ($5/$7) > "pic1.txt"
        NoFoundRatioSum+=($5/$7)
        NotFoundnum+=1
        chunknum+=1

    } 
    else if ($3 == "Bad") {
        OdessSum += $6
        BFSum2 += $7
        print "Bad " ($6/$7) > "pic1.txt"
        BadRatioSum+=($6/$7)
        Badnum+=1
        chunknum+=1

    }
    else if($2 == "Base") {
    print "Base " "1" > "pic1.txt"
    Basenum+=1
    chunknum+=1
    }
    else if($3 == "Delta") {
    print "Delta " "1" > "pic1.txt"
    Deltanum+=1
    chunknum+=1
    }    
}
END {
    print "Lz4Sum: " Lz4Sum ", BFSum1: " BFSum1 ", OdessSum: " OdessSum ", BFSum2: " BFSum2 >pic2.txt
    print "NotFoundnum: " NotFoundnum ", Badnum: " Badnum ", Basenum: " Basenum ", Deltanum: " Deltanum, "chunknum:" chunknum >pic2.txt
    print  "NotFoundnum Proportion: " NotFoundnum/chunknum " Badnum Proportion: " Badnum/chunknum " Basenum Proportion: " Basenum/chunknum " Deltanum Proportion: " Deltanum/chunknum >pic2.txt
    print "EveryNoFoundRatio: " NoFoundRatioSum/NotFoundnum " EveryBadRatio: " BadRatioSum/Badnum >pic2.txt
}
' *bf-log*
```

