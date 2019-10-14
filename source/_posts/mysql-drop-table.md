---
title: mysql删除表方式及区别
---

### mysql删除表方式及区别

> 本文记录一下这2种操作模式的区别，目标对象是表wp_comments，里面的所有留言均是垃圾留言，均可删除。然后便有了以下2种方式（进入mysql操作界面后）：

```sql
truncate table wp_comments;
delete * from wp_comments;
```
> 其中truncate操作中的table可以省略，delete操作中的*可以省略。这两者都是将wp_comments表中数据清空，不过也是有区别的，如下：

- truncate是整体删除（速度较快）， delete是逐条删除（速度较慢）。
- truncate不写服务器log，delete写服务器log，也就是truncate效率比delete高的原因。
- truncate不激活trigger(触发器)，但是会重置Identity（标识列、自增字段），相当于自增列会被置为初始值，又重新从1开始记录，而不是接着原来的ID数。而delete删除以后，Identity依旧是接着被删除的最近的那一条记录ID加1后进行记录。
> 如果只需删除表中的部分记录，只能使用DELETE语句配合where条件。 DELETE FROM wp_comments WHERE……