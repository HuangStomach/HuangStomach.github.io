---
title: SQL与优化
date: 2018-05-29 21:09:38
tags: 
- SQL
categories:
- SQL
---
> 在大多数的sql学习中，众多学习者(包括我)，可能非常容易在学习中浅尝辄止，满足于基础的增删改查，进而往往忽略了sql为我们提供的更多高效的特性，从而无论在开发还是使用上都会造成不便

## HAVING 子句

在日常使用中，使用较多的一种数据处理形式就是数据聚合，数据聚合令人第一时间想到的就是`GROUP BY`语句，具体使用可能如下:

``` sql
SELECT COUNT(*) AS count，equipment_id 
FROM eq_record
GROUP BY equipment_id
```

这种简单的使用可以非常简单的帮我们归纳数据，获取例如获取每种数据在特定情形下出现过多少次

如果想要获取到出现次数大于1的数据集合呢?我反正是这样写过:

``` sql
SELECT * FROM (
    SELECT COUNT(*) AS count，equipment_id 
    FROM eq_record
    GROUP BY equipment_id
) AS T1
WHERE count > 1
```

虽然可以得到想要的结果，不过效率非常的低，括号中的语句形成了一个子查询，我们还要再对子查询进行条件查询，效率很低

但是实际上，我们可以使用`HAVING`子句:

``` sql
SELECT COUNT(*) AS count，equipment_id 
FROM eq_record
GROUP BY equipment_id
HAVING count > 1
```

这样写不仅仅优雅，并且还去掉了子查询，增加了查询的效率

## NULL 的陷阱

很多的编程语言中都会包含布尔类型(`bool` 或者是 `boolean`)，sql中也存在着相似的概念然而，sql中还包含着第三个值`unknown`，这被称为三值逻辑

当我们进行如下查询的时候，会返回给我们失败的结果:

``` sql
SELECT *
FROM eq_record
WHERE dtend = NULL
```

这必然失败，但是为什么呢?大家肯定知道正确的写法:

``` sql
SELECT * 
FROM eq_record
WHERE dtend IS NULL
```

之所以上面的语句无法正确执行，是因为`WHERE`子句实际上只会将条件中返回结果为`true`的行加入查询结果，但是对`NULL`的比较，均会返回`unknown`

举个例子，如果我们想获取全部的使用记录(虽然我们肯定不会这么写):

``` sql
SELECT * 
FROM eq_record
WHERE dtend <> 1514736000
OR dtend = 1514736000
```

实际上`dtend`为`NULL`的行是无法返回的，任何查询执行到`NULL`行的时候都会做类似如下转换:

``` sql
SELECT * 
FROM eq_record
WHERE unknown
OR unknown
```

所以我们需要加一个条件

``` sql
SELECT * 
FROM eq_record
WHERE dtend <> 1514736000
OR dtend = 1514736000
OR dtend IS NULL
```

甚至我们在`CASE`表达式里也需要改变，如下的语句是无法执行的:

``` sql
CASE dtend
    WHEN 1      THEN 'ok'
    WHEN NULL   THEN 'no'
END
```

改变为:

``` sql
CASE WHEN dtend = 1 THEN 'ok'
     WHEN dtend IS NULL THEN 'no'
END
```

经过这么多的例子，其实可以这样假定:
**在允许的情况下，应当尽量避免`NULL`的使用**

## EXISTS 谓词

无论在哪些地方，都能听到类似的结论:

> 能用`EXISTS`就不要用`IN`

的确，比如:

``` sql
SELECT * 
FROM eq_reserv
WHERE id IN (
    SELECT reserv_id
    FROM eq_record
    WHERE dtend = 0
)
```

这样的语句实际上执行效率很慢，`IN`会产生一个子查询，每个`WHERE`条件都会去扫描子查询中的所有数据，非常耗费资源

这里我们可以使用`EXISTS `谓词:

``` sql
SELECT * 
FROM eq_reserv AS a
WHERE EXISTS (
    SELECT *
    FROM eq_record AS b
    WHERE a.id = b.reserv_id
)
```

这样比`IN`更快，如果我们在关联的列`id`上建立了索引，那么查询的时候不会查询实际的表，单独查询索引就可以了

如果使用了`EXISTS`那么不用像`IN`一样扫描全表，而是只要有一行满足条件则进行返回，比`IN`的效率高出很多

`EXISTS`与`IN`不太一样，更像是简单的做关联，如果能做出关联则进行结果的返回

## 提高性能

### 避免排序

SQL中很大部分的内存消耗实际上都发生在排序中，虽然可能没有并且的使用`ORDER BY`，但是数据库内部仍然会进行隐式的排序，如使用:

``` sql
GROUP BY
ORDER BY
SUM() COUNT() AVG() MAX() MIN()
DISTINCT
UNION INTERSECT EXCEPT
```

等等，都会触发排序，所以在提高性能的过程中，应当尽量避免使用排序，例如在使用`UNION`等的过程中，SQL会为了去除重复数据而进行排序，当我们知道不会有数据重复时，其实可以使用`UNION ALL`来避免排序

`EXISTS`的工作实际上也可以为我们去除重复数据，某种情况下我们可以使用其来代替`DISTINCT`从而避免排序

另外当我们需要使用极值函数(`MAX() MIN()`)时，使用索引能避免进行排序

### WHERE与HAVING

当使用了`HAVING`子句的时候，可能会有这样的困惑，既然`HAVING`可以为我们提升聚合的效率，是不是应该多使用`HAVING`?

实际上并不全是如此，如果条件允许的情况下，能写在`WHERE`子句中的条件应该尽量写在`WHERE`子句中，这样在`GROUP BY`进行聚合排序的时候才会降低排序的负担，从而提高性能，并且很多聚合后的视图是不会保有原本的索引的

### 正确使用索引

众所周知索引是能为我们大大提高查询效率的，但是很多时候我们并没有正确使用索引:

``` sql
SELECT * 
FROM eq_record
WHERE dtend * 1.5 < 1514736000
```

上这里对索引进行了运算，然而这种转换数据库是不会帮我们做的，所以我们可以这样写:

``` sql
SELECT * 
FROM eq_record
WHERE dtend < 1514736000 / 1.5
```

当使用索引的时候，条件表达式左侧应该尽量是原始字段，另外，使用`NULL`来做判断的时候，也是无法使用索引的

有些时候我们会用到联合索引，比如`(user_id，equipment_id)`，这样的时候使用顺序则很重要

``` sql
SELECT * FROM eq_record WHERE user_id = 1 AND equipment_id = 1
SELECT * FROM eq_record WHERE equipment_id = 1 AND user_id = 1
```

第二条语句仅仅是颠倒了顺序，就使得数据库无法充分使用索引从而变慢，所以当使用联合索引时应尽量让顺序和定义索引的时候保持一致

### 使用IN

有些时候我们迫不得已会使用`IN`谓词例如:

``` sql
SELECT * 
FROM eq_sample
WHERE sender_id IN (SELECT id FROM user WHERE atime > 0)
AND lab_id IN (SELECT lab_id FROM user WHERE atime > 0)
```

我们这里使用了两个子查询(先不考虑是否可以转换使用`EXISTS`谓词)，并且子查询的内容是一样的，如果我们这么执行，则会生成两个完全相同的子查询集合，所以我们可以这样写:

``` sql
SELECT *
FROM eq_sample
WHERE (sender_id, lab_id)
IN (SELECT id, lab_id FROM user WHERE atime > 0)
```

则会让sql重复使用一个子查询，尽量提高效率