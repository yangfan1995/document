# Chapter 5 数据读取与存储

## RDD

It's important to note that during the application development, you can write code, compile it, and even run your job, and unless you materialize the RDD, your code may not have even tried to load the original data.(惰性，单纯的表明谱系图)

## 读取数据到RDD

```scala
val dataRDD = sc.parallelize(List(1,2,4))
```

The collect() function only works if your data fits
in memory on a single host; in that case it adds the bottleneck of everything having
to come back to a single machine.(collect()函数的调用时，如果内存中放不下会产生内存溢出，成为性能瓶颈)

## RDD操作

broadcast and accumulator variables.

