elasticsearch
===
Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档，用JSON作为文档序列化的格式。

其结构相对于关系数据库的对比如下：

关系数据库 ⇒ 数据库 ⇒ 表 ⇒ 行 ⇒ 列(Columns)

Elasticsearch ⇒ 索引 ⇒ 类型 ⇒ 文档 ⇒ 字段(Fields)

### lucene全文检索

### 倒排索引

### Posting List
Elasticsearch分别为每个field都建立了一个倒排索引，Posting list就是一个int的数组，存储了所有符合某个term的文档id。

### Term Dictionary
Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效率，就像通过字典查找一样，这就是Term Dictionary。

### Term Index
 B-Tree通过减少磁盘寻道次数来提高查询性能，Elasticsearch也是采用同样的思路，直接通过内存查找term，不读磁盘，
 但是如果term太多，term dictionary也会很大，放内存不现实，
 于是有了Term Index，就像字典里的索引页一样，A开头的有哪些term，分别在哪页，可以理解term index是一颗树：
 
![session](https://github.com/IceDarron/Note/blob/master/Image/elasticsearch_term_index.png)

这棵树不会包含所有的term，它包含的是term的一些前缀。
通过term index可以快速地定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。

### 二元搜索算法
二元搜索算法是在排好序的数组中找到特定的元素

首先, 比较数组中间的元素,如果相同,则返回此元素的指针,表示找到了. 
如果不相同, 此函数就会继续搜索其中大小相符的一半,然后继续下去. 
如果剩下的数组长度为0, 则表示找不到,函数就会结束.

此算法函数如下:

```java
int *binarySearch(int val, int array[], int n)
{
    int m = n/2;
    if(n <= 0) return NULL;
    if(val == array[m]) return array + m;
    if(val < array[m]) return binarySearch(val, array, m);
    else return binarySearch(val, array+m+1, n-m-1);
}
```

对于有n个元素的数组来说,二元搜索算法进行最多1+log2(n)次比较. 如果有一百万元素,最比较20次, 也就是最多20次递归执行binarySearch()函数.


### 压缩算法
首先，对词典文件中的关键词进行了压缩，关键词压缩为<前缀长度，后缀>，
例如：当前词为“阿拉伯语”，上一个词为“阿拉伯”，那么“阿拉伯语”压缩为<3，语>。

ab				(vint) 0 ab

abcd			(vint) 2 cd

abcde			(vint) 4 e

其次大量用到的是对数字的压缩，数字只保存与上一个值的差值（这样可以减少数字的长度，进而减少保存该数字需要的字节数）。
例如当前文章号是16389（不压缩要用3个字节保存），上一文章号是16382，压缩后保存7（只用一个字节）。