**Revisiting Frequency Analysis against Encrypted Deduplication via Statistical Distribution**

#### Abstract
加密重复数据删除的确定性可能会泄露明文频率，使其容易受到频率分析攻击，基于频率分布的攻击可以推断出密文-明文对 。
本文介绍了一种基于分布的攻击，该攻击利用统计建模来提高推断密文-明文对的精度，特别是针对加密重复数据删除中的备份工作负载，并过滤掉**虚假推断**。

#### How to filter out false inferences
该攻击可以配置参数 “r” 来定义每种唯一密文要检查的明文范围，从而提供抵御密文和明文频率排序干扰的能力 。 通过适当设置 “r”，攻击可以将每个唯一的密文与定义范围内的明文进行匹配，从而确保频率排名的偏差不会显著影响攻击准确推断密文-明文对的能力