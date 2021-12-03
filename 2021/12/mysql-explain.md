# Mysql - explain 使用

## 执行计划包含的字段
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra |
| ---- | ----------- | ----- | ---- | ------------- | ---- | ------- | ---- | ---- | ----- |
|      |             |       |      |               |      |         |      |      |       |

##  各字段含义
### id（查看是否是小表驱动大表）
select查询的序列号，包含一组数字，表示查询中执行select字句或操作表的顺序  
  + id相同，执行顺序从上到下  
  + id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行  
  + id相同和不同，同时存在，id值大的优先执行，id值相同的从上到下执行

### select_type
  + SIMPLE, 简单的select查询，查询中不包含子查询或UNION
  + PRIMARY,  查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
  + SUBQUERY, 在select或where列表中包含了子查询（内部查询）
  + DERIVED, 在from列表中包含的子查询被标记为DERIVED（衍生）,Mysql会递归执行这些子查询，把结果放在临时表中
  + UNION, 若第二个select查询出现在UNION后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外侧select将被标记为DERIVED
  + UNION RESULT, 从UNION的合并
### table
指的是当前执行的表
### type
type表示查询是使用的哪种类型  
	system > const > eq_ref > ref > range > index > all
一般来说，得保证查询至少达到range级别，最好能达到ref
  + system  
    system表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略
  + const  
    const表示通过索引一次就能找到，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换成一个常量。
  + eq_ref(唯一性索引扫描)  
    对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一性索引扫描。
  + ref(非唯一性索引扫描)  
    返回匹配某个单独值得所有行，本质上也是索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体
  + range(范围扫描索引)  
    使用一个索引来选择行，key列显示使用了哪个索引，一般就是在你的where语句中出现between, < , >, in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。
  + index(遍历索引)  
    Index与All区别为index类型只遍历索引树。all和index都是读全表，但index是从索引中读取的，all是从硬盘读取的。
  + all(全表扫描)  
		将遍历全表以找到匹配的行。

### possible_keys
possible_keys显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。
### key
查询中若使用了**覆盖索引**（select后要查询的字段刚好和创建的索引字段安全相同），则该索引仅出现在key列表中。
### key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度，在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的。
### ref
显示索引的那一列被使用了，如果可能的话，最好是一个常数。
### rows
大致估算出找到所需的记录所需要读取的行数（数值越少越好 ）
### Extra
包含不适合在其他列中显示但是十分重要的额外信息。
	+ Using filesort(无法按索引进行排序，按文件排序)
		说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排序”。
	+ Using temporary(新建临时表，拖慢sql)
		使用了用临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by。
	+ Using index(使用了覆盖索引covering index，避免了表扫描查询)
		表示相应的select操作中使用了覆盖索引（Covering Index），避免访问了表的数据行，效率不错。如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。为了达到**索引覆盖**的效果，select的列只取出索引中的列。
	+ Using join buffer
		表明使用了连接缓存,比如说在查询的时候，多表join的次数非常多，那么将配置文件中的缓冲区的join buffer调大一些。
	+ impossible where
		where子句的值总是false，不能用来获取任何元组
		
# Reference
+ https://zhuanlan.zhihu.com/p/93425047
