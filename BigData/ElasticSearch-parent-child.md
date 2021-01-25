# ElasticSearch的Parent-Child



Parent-Child与Nested非常类似，都可以用来处理一对多的关系，如果是多对多的关系，那就拆分成一对多之后，再进行处理。

nested的缺点是对数据的更新需要reindex整个nested结构下的所有数据，所以注定了它的使用场景一定是查询多更新少的场景，如果是更新多的场景，那么nested的性能未必会很好。

而parent-child就非常适合在更新多的场景，因为Parent-Child的数据存储都是独立的，只要求父子文档都分布在同一个shard里面即可。而nested模式下，不仅要求同一个shard下还是必须在同一个segment里面的同一个block下，这种模式注定了nested的查询性能要比Parent-Child好，但是更新性就大大不如Parent-Child了，对比nested模式，Parent-Child主要有下面几个特点：

1. 父文档可以被更新，而无需重建所有的子文档
2. 子文档的添加，修改或者删除不影响它的父文档和其他的子文档，这尤其是在子文档数据巨大而且需要被添加和更新频繁的场景下Parent-Child能获得更好的性能
3. 子文档可以被返回在搜索结果里面



https://www.elastic.co/guide/en/elasticsearch/reference/5.0/mapping-parent-field.html

该特性在6.0之后已经删除了。

转移到了https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html



事实上，ES作为搜素引擎，他能力体现在搜索引擎之上，而不是关系性。



## 参考文档

https://blog.csdn.net/u010454030/article/details/77840606