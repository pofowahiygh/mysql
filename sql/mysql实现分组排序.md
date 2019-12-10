如下stu表展示各班级学生的分数，如何实现按照班级对学生成绩进行排序，取每个班前三名？

两种方法，先说第一种：


id | class_name | stu_id | point
--- | --- | --- | ---
1	|classOne	|1001	|89
2	|classOne	|1002	|98
3	|classOne	|1003	|59
4	|classOne	|1004	|76
5	|classOne	|1005	|98
6	|classTwo	|2001	|76
7	|classTwo	|2002	|89
8	|classTwo	|2003	|59
9	|classTwo	|2004	|93
10	|classTwo	|2005	|88
11	|classThree	|3001	|89
12	|classThree	|3002	|54
13	|classThree	|3003	|78
14	|classThree	|3004	|70
15	|classThree	|3005	|87
17	|七班		|1001	|99
18	|七班		|1001	|99
19	|七班		|1001	|99
……|……|……|……

先上sql"

```sql
select
	a.id,
    a.class_name,
    a.stu_id,
    a.point
from stu a 
left join stu b on a.class_name = b.class_name
	and a.point < b.point
group by a.id,
		a.class_name,
        a.stu_id,
        a.point
having count(b.id) <= 2
order by a.class_name, a.point desc
limit 100;

```

下面分步解释：
现将sql整理成如下形式(只观察一个班 classOne)
```sql
select
	a.*,b.*
from stu a 
left join stu b on a.class_name = b.class_name
	and a.point < b.point
where a.class_name = 'classOne'
order by a.point asc;
```
返回结果如下：

a.id | a.class_name| a.stu_id | a.point | b.id | b.class_name| b.stu_id | b.point
--- | --- |--- | ---|--- | ---|--- | ---
3	|classOne	|1003	|59	|2	|classOne	|1002	|98
3	|classOne	|1003	|59	|4	|classOne	|1004	|76
3	|classOne	|1003	|59	|5	|classOne	|1005	|98
3	|classOne	|1003	|59	|1	|classOne	|1001	|89
4	|classOne	|1004	|76	|5	|classOne	|1005	|98
4	|classOne	|1004	|76	|1	|classOne	|1001	|89
4	|classOne	|1004	|76	|2	|classOne	|1002	|98
1	|classOne	|1001	|89	|5	|classOne	|1005	|98
1	|classOne	|1001	|89	|2	|classOne	|1002	|98
5	|classOne	|1005	|98				
2	|classOne	|1002	|98	

可以看到，分数最低的59，关联到了4条比它大的记录，而76关联到了3条比它大的，89关联到了2条比它大的，而最大分数89关联的结果为空；

这时对b表的任何一个字段进行count统计就可知道每条记录的排名，对count(b.id) 进行倒序排列，取前n项就可得到班级前n+1名；


```sql
select
	a.id,
    a.class_name,
    a.stu_id,
    a.point
from stu a 
left join stu b on a.class_name = b.class_name
	and a.point < b.point
group by a.id,
		a.class_name,
        a.stu_id,
        a.point
having count(b.id) <= 2
order by a.class_name, a.point desc
limit 100;

```
返回结果：

id | class_name | stu_id | point
--- | --- | ---| ---
5	|classOne	|1005	|98
2	|classOne	|1002	|98
1	|classOne	|1001	|89
11	|classThree	|3001	|89
15	|classThree	|3005	|87
13	|classThree	|3003	|78
9	|classTwo	|2004	|93
7	|classTwo	|2002	|89
10	|classTwo	|2005	|88
17	|七班		|1001	|99
18	|七班		|1001	|99
19	|七班		|1001	|99	




下面说第二种：
先上sql

```sql

select
    id,
    class_name,
    stu_id,
    point,
    rank
from 
(
    select
    	stu.*,
        @rank:=if(@class = class_name,@rank + 1, 1) as  rank,
        @class:=class_name
    from stu,(select @rank:=0,@class:=null) a
    order by class_name,point desc
) a 
where rank <=3;

```
结果如下：

id | class_name | stu_id | point | rank
--- | --- |  --- |  --- |  ---
2	|classOne	|1002	|98	|1
5	|classOne	|1005	|98	|2
1	|classOne	|1001	|89	|3
11	|classThree	|3001	|89	|1
15	|classThree	|3005	|87	|2
13	|classThree	|3003	|78	|3
9	|classTwo	|2004	|93	|1
7	|classTwo	|2002	|89	|2
10	|classTwo	|2005	|88	|3
17	|七班		|1001	|99	|1
18	|七班		|1001	|99	|2
19	|七班		|1001	|99	|3


参考资料：http://blog.sina.com.cn/s/blog_72d3486f0102w1mq.html


https://www.xaprb.com/blog/2006/12/07/how-to-select-the-firstleastmax-row-per-group-in-sql/