# Interview-doc
## 问题一

仓库地址：
https://github.com/Jackson10240/RateLimiter.git

git@github.com:Jackson10240/RateLimiter.git




## 问题二


### 1. 优化索引


#### 问题分析：
table_l 表的 flag_type_date 索引包含了 flag, type, date_end, date_start 四个字段。
table_l 表的查询条件中使用了 flag, type, date_end, date_start，这些条件都可以利用索引。
table_d 表的查询条件中使用了 id_m，但没有使用索引。

#### 优化建议：
为 table_d 表的 id_m 字段添加索引。


```SQL
ALTER TABLE table_d ADD INDEX idx_id_m (id_m);
```

#### 原因： 
添加索引可以加速 table_d 表的查询，减少全表扫描的开销。



### 2. 优化查询条件


#### 问题分析：
查询条件中使用了 DATE('2019-07-01') 和 DATE('2019-07-01') + INTERVAL 1 month，这些操作在每次查询时都会重新计算，增加了查询的开销。


#### 优化建议：
将日期计算提前到应用程序中，避免在 SQL 查询中重复计算。


```SQL
-- 假设应用程序中已经计算好了 start_date 和 end_date
SELECT 
    filed_a
FROM table_l 
JOIN table_d on table_l.id = table_d.id
WHERE
    table_l.date_end >= '2019-07-01'
    AND table_l.date_end < '2019-08-01'
    AND table_l.date_end >= table_l.date_start
    AND table_l.flag = 1
    AND table_l.type = 'S'
    AND table_d.id_m = 1234;
```

#### 原因： 
减少 SQL 查询中的计算开销，提高查询效率。



### 3. 优化表结构

#### 问题分析：
table_l 表的总数据量接近 1KW，数据量较大，可能会导致查询效率低下。

#### 优化建议：
考虑对 table_l 表进行分区，根据 date_end 字段进行分区。

```SQL
ALTER TABLE table_l PARTITION BY RANGE (TO_DAYS(date_end)) (
    PARTITION p201907 VALUES LESS THAN (TO_DAYS('2019-08-01')),
    PARTITION p201908 VALUES LESS THAN (TO_DAYS('2019-09-01')),
    -- 其他分区
);
```

#### 原因：
分区可以减少每次查询需要扫描的数据量，提高查询效率。


### 4. 优化查询语句
#### 问题分析：
查询语句中使用了 JOIN，可能会导致性能问题。

#### 优化建议：
考虑将 JOIN 操作拆分为两个独立的查询，然后在应用程序中进行合并。

```SQL
-- 第一步：查询 table_l
SELECT 
    id
FROM table_l 
WHERE
    date_end >= '2019-07-01'
    AND date_end < '2019-08-01'
    AND date_end >= date_start
    AND flag = 1
    AND type = 'S';

-- 第二步：查询 table_d
SELECT 
    filed_a
FROM table_d 
WHERE
    id_m = 1234
    AND id IN (SELECT id FROM table_l WHERE ...);

```

#### 原因：
拆分查询可以减少 JOIN 操作的开销，提高查询效率。


