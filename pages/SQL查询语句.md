- # 基本查询
	- 使用``SELECT {col_name} FROM {form_name}``可以从表``form_name``中选择列``col_name``
	- SELECT语句也能用于计算，例如``SELECT 100+200``但一般不会这样做
	- 但是``SELECT 1``常用于测试数据库连接
- # 条件查询
	- 在SELECT语句后增添``WHERE``可以指定查询条件
	- 支持与或非条件拼接，关键字分别为``AND, OR, NOT``，使用括号调整结合优先级
	- 常用条件运算符
		- | 条件 | 表达式举例1 | 表达式举例2 | 说明 |
		  | ---- | ---- | ---- |
		  | 使用=判断相等 | score = 80 | name = 'abc' | 字符串需要用单引号括起来 |
		  | 使用>判断大于 | score > 80 | name > 'abc' | 字符串比较根据ASCII码，中文字符比较根据数据库设置 |
		  | 使用>=判断大于或相等 | score >= 80 | name >= 'abc' |  |
		  | 使用<判断小于 | score < 80 | name <= 'abc' |  |
		  | 使用<=判断小于或相等 | score <= 80 | name <= 'abc' |  |
		  | 使用<>判断不相等 | score <> 80 | name <> 'abc' |  |
		  | 使用LIKE判断相似 | name LIKE 'ab%' | name LIKE '%bc%' | %表示任意字符，例如'ab%'将匹配'ab'，'abc'，'abcd' |
		  |BETWEEN AND 用于指定范围| BETWEEN 60 AND 90|||
- # 投影查询并排序
	- 即不选中所有列，在SELECT中指定要返回的列名
	- 在选中列时可以同时指定别名
		- ``SELECT {col_name_0} {alias_0}, {col_name_1} {alias_1} ...``
	- 在查询语句的最后加上``ORDER BY {col_name}``即可对查询结果**根据某列进行排序**
		- 追加``DESC``关键字即可**倒叙排序**
		- 也可以按照多列排序，和给出的顺序保持一致，如：``ORDER BY {col_name_0} DESC, {col_name_1}, ...``
		- 实际上，升序也可以显式给出，即``ASC``，但是由于默认使用升序，所以可以省略
- # 分页
	- 当查询返回的结果数据量过大时，分页将是各明智的选择
	- 在查询语句的最后加上``LIMIT {page_capacity} OFFSET {offset}``即可完成分页
		- LIMIT指定每一页显式的最多数量，offset即查询起始位置
		- 每此查询之后，下一次查询经OFFSET的值增加一个LIMIT即可获取下一页内容
		- 若LIMIT大于总的结果数据量，则返回空集
	- 在``OFFSET``如果不给出，则默认为0
	- [[$red]]==在MySql中==，可简写为``LIMIT {paga_capacity}, {offset}``
	- [[$red]]==**注意**==：随着offset的增加，查询效率会越来越低
- # 聚合查询
	- 使用关键字``COUNT``可以快速统计某一列的数据量，例如
		- ``SELECT COUNT(*) num FROM students``
		- num是给出结果的别名
		- 查询结果是一个[[$red]]==**只有一列一行的表**==，唯一的列名称为``num``
		- [[$red]]==**并不会去重**==
	- 还有一些常见的**聚合函数**
		- | 函数 | 说明 |
		  | ---- | ---- | ---- |
		  | SUM | 计算某一列的合计值，该列必须为数值类型 |
		  | AVG | 计算某一列的平均值，该列必须为数值类型 |
		  | MAX | 计算某一列的最大值 |
		  | MIN | 计算某一列的最小值 |
	- 在使用聚合查询时，若通过``WHERE``过滤之后没有任何数据，则``COUNT``会返回0，另外给出的四个聚合函数会返回``NULL``
	- ## 分组聚合
		- 通过``GROUP BY``关键字可以分组聚合，一次性完成多个操作，例如``SELECT COUNT(*) num FROM studetns GROUP BY class_id``
			- 此查询语句可以一次性获取不同班级学生的人数
		- 为了可读性，一般建议将聚合的列也加入输出列，例如:
			- ``SELECT class_id COUNT(*) num FROM students GROUP BY class_id``
		- 在聚合时，不能输出在某一分组中元素不相同的列
		- 可以同时在多个列上进行聚合，不同列之间用`,`分割
- ## 多表查询
	- 在``FROM``后可以有多张表，用`,`分割，如此查询出来的是两张表的**[[$red]]==笛卡尔乘积==**，例如：
		- ```sql
		  SELECT
		      s.id sid,
		      s.name,
		      s.gender,
		      s.score,
		      c.id cid,
		      c.name cname
		  FROM students s, classes c;
		  ```
	- ## 连接查询
		- 选定一个主表作为结果集，然后将其他表的行为有选择性地连接在主表上
		- ### 内连接
			- 例如：
			- ```sql
			  SELECT s.id sid, s.name sname, s.score, c.name
			  FROM students s
			  INNER JOIN classes c ON s.class_id = c.id
			  WHERE s.score BETWEEN 60 AND 90;
			  ```
			- 内连接获的结果一定是**[[$red]]==两张表中都有数据==**
		- ### 外连接
			- 分为``LEFT OUTER JOIN``，``RIGHT OUTER JOIN``和``FULL OUTER JOIN``
			- 左外连接会返回左表存在但右表不存在的值，右外连接会返回右表存在但左表不存在的值，全连接会返回任何数据
- # 高级
	- ## 子查询
		- ```sql
		  SELECT column_name [, column_name ]
		  FROM   table1 [, table2 ]
		  WHERE  column_name OPERATOR
		        (SELECT column_name [, column_name ]
		        FROM table1 [, table2 ]
		        [WHERE])
		  ```