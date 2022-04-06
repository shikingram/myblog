---
title: "MySQL防止重复插入相同记录 insert if not exists"   
author: "Kingram"  
date: 2022-04-06   
lastmod: 2022-04-06

tags: [  
"mysql",
]
---
看下如下的语句：
```sql
INSERT INTO TABLE (name, age, sex) SELECT
	'kingram',
	'19',
	'male'
FROM
	DUAL
WHERE
	NOT EXISTS (
		SELECT
			*
		FROM
			TABLE
		WHERE
			name = 'kingram'?
	)
```
这条语句的作用是防止插入两条一模一样的name为kingram的记录，这里用的语法是

```sql
insert into select
```
这个语句是复制表的意思，我们可以从一个表中复制所有的列插入到另一个已存在的表中：
```sql
INSERT INTO table2
SELECT * FROM table1;
```
或者我们可以只复制希望的列插入到另一个已存在的表中：
```sql
INSERT INTO table2
(column_name(s))
SELECT column_name(s)
FROM table1;
```

这样这句sql就很好理解了，我们从dual表中复制一份记录到目标table中，条件是这条记录的name不为kingram.

### 使用场景
这样的sql适合用在表中预插入数据时，例如在进行数据库迁移的时候，表中需要预插入数据，那么就可以在迁移脚本中写一条这样的数据，这样无论表中有没有数据都能保证最终存在这样的记录，而且不会报错。