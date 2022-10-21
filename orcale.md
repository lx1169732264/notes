



DUAL	最小的工作表，只有一行一列.通常用于计算常数时指定的表名



MERGE INTO [target-table] A USING [source-table sql] C ON([conditional expression] and […]…)
WHEN MATCHED THEN
[UPDATE sql]
WHEN NOT MATCHED THEN
[[INSERT](https://so.csdn.net/so/search?q=INSERT&spm=1001.2101.3001.7020) sql]

判断A表和C表是否满足某条件，如果满足则用C表去更新A表，如果不满足，则将C表数据插入A表





# varchar2

1.varchar是标准sql里的。 varchar**2**是**oracle**的**独有**数据类型。

2.varchar汉字两字节，英文一字节，varchar2是GBK编码,都占两字节

3.varchar对空串不处理，varchar2将空串当做**null**来处理

4.varchar定长，最大长度是**2000**，varchar**2**是存放可变长度的字符串，最大长度是**4000**.



## 时间日期

只有DATE类型，包含年月日时分秒
日期字段的数学运算公式有很大的不同。MYSQL为SUBDATE（NOW（），INTERVAL 7 DAY）ORACLE为SYSDATE - 7;



# rownum

rownum默认按数据**插入时间倒序**排列

虚列,不需要指定表名	`t.nownum`会报错

rownum的**赋值是在数据库解析完查询语句后，在查询语句做排序或聚合函数执行之前** -> 先有结果表,再有rownum. 除了分页以外, rownum一般不作为查询条件

```sql
select * from dept where rownum>1; 
##输出0条, 因为rownum起始为1
```



## 分页

子查询内层加上rownum, 外层分页

```sql
select * from (
  select rownum, t.* from t 
) where rownum between 1 and 10;
```



## 排序分页

先排序, 再加rownum, 最后根据rownum分页

```sql
select * from (
  select rownum, tmp.* from
    ( select * from t order by c desc ) tmp
) where rownum between 1 and 10;
```

















