### Hive_sql如何计算连续签到天数？
##### 在a表中有id和date两列，记录用户当天是否签到，想查询出哪些用户连续签到了3天（或连续签到更多天，是连续签到），sql改如何写呢？


a 表结果如下

id | date
---|---
1 | 2017-01-01
1 | 2017-01-02
1 | 2017-01-03
2 | 2017-01-01
2 | 2017-01-02
2 | 2017-01-03
3 | 2017-01-01
3 | 2017-01-02
4 | 2017-01-01


知乎大神如此回答：

用row_number来判断就行了。

以下是详细说明：

```sql
select
    id,
    date,
    row_number() over(partition by id order by date) as rank
from a 
```

查询结果如下：b表


id | date | rank
---|---|---
1 | 2017-01-01 | 1
1 | 2017-01-02 | 2
1 | 2017-01-03 | 3
2 | 2017-01-01 | 1
2 | 2017-01-02 | 2
2 | 2017-01-03 | 3

……

设置一个新字段  date_rank （名字可以随便叫）
再count 不同的date_rank 作为cnt ，并用 group by id,(date_rank)

```sql 
-- c 表
select
    id,
    date_sub(date, rank) as date_rank
from b
```

```sql
select
    id,
    date_rank,
    count(distinct date_rank) as cnt
from c
group by id,date_rank
```




得到结果如下：date_rank 只是一个辅助字段，cnt为连续签到的天数


id | date_rank | cnt
---|---|---
1 | 2016-12-31 | 3
2 | 2016-12-31 | 3
……
