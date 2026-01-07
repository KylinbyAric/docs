#特性
##分布式扩展
##全文检索
##实时的搜索和分析
##高可用
##模式自由



问题：
1、怎样插入一条文档的？
2、怎样查询一条文档的？

#基础概念
##索引
类似mysql中的数据库概念
##类型
类似mysql中的表，用户定义数据类型--存疑
即定义文档中的key是哪些类型的。
有Keyword、text、integer类型
keyword:是直接建立反向索引
test：是先分词，然后用分词建立索引
##文档
es的数据行是一个文档，即一个JSON格式的字符串。
##
##倒排索引
es会将所有文档中的关键字term都建立一个索引<关键字,postinglist>。这样可以通过倒排索引快速查找关键字对应的文档id。
postinglist是指文档id、次数等信息
##Term Dictionary
关键字term合起来就是一个Term Dictionary。存储结构是有序二叉排序树--存疑，为什么不是多叉。
##Term index
Term Dictionary过大导致内存放不下，那就建立一个字典树，通过字典树快速找到term。term index不需要存储所有term，只需要存储前缀和term block即可
![三次握手](../img/es_term_index.png)
##mapping
一个mapping由一个或多个analyzer组成， 一个analyzer又由一个或多个filter组成的。
在存储内容时，是会先用mapping进行处理，把内容解析分割成各个索引字符，然后存储起来。
在查询内容时，是先用mapping进行处理，把内容解析分割成各个索引字符，然后带着这些索引字符去查找。
如果想某个字符不被filter掉，能保存成索引。那就需要改mapping。
分词的功能应该就是用这个实现的

