Q:MapReduce
===
MapReduce其实是分治算法的一种实现，所谓分治算法就是“就是分而治之”，将大的问题分解为相同类型的子问题（最好具有相同的规模），对子问题进行求解，然后合并成大问题的解。MapReduce就是分治法的一种，将输入进行分片，然后交给不同的task进行处理，然后合并成最终的解。

![mapreduce模型](https://github.com/IceDarron/Note/blob/master/Image/mapReduce_model.png)

![mapreduce模型](https://github.com/IceDarron/Note/blob/master/Image/mapReduce_model0.png)

### Map阶段
首先是读数据，数据来源可能是文本文件，表格，MySQL数据库。这些数据通常是成千上万的文件（叫做shards），这些shards被当做一个逻辑输入源。然后Map阶段调用用户实现的函数，叫做Mapper，独立且并行的处理每个shard。对于每个shard，Mapper返回多个键值对，这是Map阶段的输出。

### Shuffle阶段
把键值对进行归类，也就是把所有相同的键的键值对归为一类。这个步骤的输出是不同的键和该键的对应的值的数据流。

### Reduce阶段
输入当然是shuffle的输出。然后Reduce阶段调用用户实现的函数，叫做Reducer，对每个不同的键和该键的对应的值的数据流进行独立、并行的处理。每个reducer遍历键对应的值，然后对值进行“置换”。这些置换通常指的的是值的聚合或者什么也不处理，然后把键值对写入数据库、表格或者文件中。

本问题仅简单介绍，相关资料如下：http://blog.csdn.net/u010725690/article/details/53893667
