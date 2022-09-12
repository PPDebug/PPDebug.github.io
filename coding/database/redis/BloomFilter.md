# Redis布隆过滤器

> 布隆过滤器（Bloom Filter）被作为插件加载到 Redis 服务器中，给 Redis 提供强大的去重功能。

相较于Set集合的去重功能而言，过滤器在空间上节省90%以上，但去重率在99%左右。

## 应用场景

1. 百度爬虫系统每天会面临海量的 URL 数据，希望它每次只爬取最新的页面，而对于没有更新过的页面则不爬取，因策爬虫系统必须对已经抓取过的 URL 去重，否则会严重影响执行效率。但是如果使用一个 set（集合）去装载这些 URL 地址，那么将造成资源空间的严重浪费。
2. 垃圾邮件过滤功能也采用了布隆过滤器。虽然在过滤的过程中，布隆过滤器会存在一定的误判，但比较于牺牲宝贵的性能和空间来说，这一点误判是微不足道的。

## 工作原理

[Bloom Filter Diagram](./assets/BloomFilter.drawio ":include :type=code")


当使用布隆过滤器添加 key 时，会使用不同的 hash 函数对 key 存储的元素值进行哈希计算，从而会得到多个哈希值。根据哈希值计算出一个整数索引值，将该索引值与位数组长度做取余运算，最终得到一个位数组位置，并将该位置的值变为 1。每个 hash 函数都会计算出一个不同的位置，然后把数组中与之对应的位置变为 1。通过上述过程就完成了元素添加(add)操作。

需要判断一个元素是否存时，其流程如下：首先对给定元素再次执行哈希计算，得到与添加元素时相同的位数组位置，判断所得位置是否都为 1，如果其中有一个为 0，那么说明元素不存在，若都为 1，则说明元素有可能存在。

## 失效原因

[Bloom Filter Diagram](./assets/BloomFilterError.drawio ":include :type=code")


```conf
key  #指定存储元素的键，若已经存在,则bf.reserve会报错
error_rate=0.01 #表示错误率
initial_size=100 #表示预计放入布隆过滤器中的元素数量
```
当放入过滤器中的元素数量超过了 initial_size 值时，错误率 error_rate 就会升高。因此就需要设置一个较大 initial_size 值，避免因数量超出导致的错误率上升。

错误率越低，所需要的空间也会越大，因此就需要我们尽可能精确的估算元素数量，避免空间的浪费。我们也要根据具体的业务来确定错误率的许可范围，对于不需要太精确的业务场景，错误率稍微设置大一点也可以。


