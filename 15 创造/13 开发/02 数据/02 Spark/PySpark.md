---
title: PySpark
toc: true
date: 2018-06-11 08:14:30
---

## 缘由：

简单的记一下，暂时没有深入

## 要点：


代码如下：


```py
import sys
from operator import add
from pyspark import SparkContext
sc = SparkContext()


lines = sc.textFile("stormofswords.csv")
counts = lines.flatMap(lambda x: x.split(',')) \
              .map(lambda x: (x, 1)) \
              .reduceByKey(add)
output = counts.collect()
output = filter(lambda x:not x[0].isnumeric(), sorted(output, key=lambda x:x[1], reverse = True))
for (word, count) in output[:10]:
    print "%s: %i" % (word, count)

sc.stop()
```

<span style="color:red;">没有自己执行过，要学习下，这里只是简单的记录下视频中的代码</span>

输出应该是：


```
Tyrion: 36
Jon: 26
Sansa: 26
Robb: 25
Jaime: 24
Tywin: 22
Cersei: 20
Arya: 19
Robert: 18
Joffrey: 18
```
