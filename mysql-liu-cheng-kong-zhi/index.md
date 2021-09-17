# 【MySQL】流程控制


## 结构分类

流程控制的结构，可分为如下几类：

- 顺序结构：程序从上往下依次执行。

- 分支结构：从多条路径中选择一条去执行。

- 循环结构：在满足一定条件的基础上，重复执行一段代码。


## 顺序结构

`SQL`语句默认就是从上往下依次执行。


## 分支结构

分支结构主要有以下几种语法：

- `IF`函数

- `CASE`结构

- `IF`结构

### 1. `IF`函数

```sql
IF(表达式1，表达式2，表达式3)
```

如果表达式1成立，则IF函数返回表达式2的值，否则返回表达式3的值。

### 2. `CASE`结构

```sql
CASE 变量|表达式|字段
WHEN 要判断的值 THEN 返回的值1或语句;
WHEN 要判断的值 THEN 返回的值2或语句;
...
ELSE 返回的值或语句;
END CASE;
```

或者

```sql
CASE
WHEN 要判断的条件1 THEN 返回的值1或语句;
WHEN 要判断的条件2 THEN 返回的值2或语句;
...
ELSE 要返回的值;
END CASE;
```

以上两种写法主要用于`BEGIN/END`语句块中。其他情况下，可以使用如下语句：

```sql
CASE 变量|表达式|字段
WHEN 要判断的值 THEN 返回的值1或语句
WHEN 要判断的值 THEN 返回的值2或语句
...
ELSE 返回的值或语句
END
```

或者：

```sql
CASE
WHEN 要判断的条件1 THEN 返回的值1或语句
WHEN 要判断的条件2 THEN 返回的值2或语句
...
ELSE 要返回的值
END
```

### 3. `IF`结构

```sql
IF 条件1 THEN 语句1;
ELSEIF 条件2 THEN 语句2;
...
ELSE 语句n;
END IF;
```

IF结构主要应用于`BEGIN/END`语句块中。

### 4. 示例

- `CASE`结构

```mysql
DELIMITER $
CREATE PROCEDURE test_case(IN score int)
BEGIN
    CASE
    WHEN score>=90 AND score<=100 THEN SELECT 'A';
    WHEN score>=80 THEN SELECT 'B';
    WHEN score>=60 THEN SELECT 'C';
    ELSE SELECT 'D';
    END CASE;
END $
```

- `IF`结构

```mysql
DELIMITER $
CREATE FUNCTION test_if(score INT) RETURNS CHAR
BEGIN
    IF score>=90 AND score<=100 THEN RETURN 'A';
    ELSEIF score>=80 THEN RETURN 'B';
    ELSE RETURN 'D';
    END IF;
END $
```


## 循环结构

循环结构主要有以下几种语法：

- `WHILE`

- `LOOP`

- `REPEAT`

控制循环流程的语法如下：

- `ITERATE`：类似于`continue`。

- `LEAVE`：类似于`break`。

### 1. 循环结构语法

```sql
[标签:]WHILE 循环条件 DO
循环体;
END WHILE [标签];
```

```sql
[标签:]LOOP
循环体;
END LOOP [标签];
```

```sql
[标签:]REPEAT
循环体;
UNTIL 结束循环的条件
END REPEAT [标签];
```

注意`LOOP`需要搭配标签使用，否则会变成死循环。

标签主要用于搭配`iterate`和`leave`跳出循环。

### 2. 示例

- `WHILE`

```sql
DELIMITER $
CREATE PROCEDURE test_while(n int)
BEGIN
    declare sum int default 0;
    WHILE n > 0 DO
        SET sum=sum+n;
        SET n=n-1;
    END WHILE;
    SELECT sum AS result;
END$
```

- `REPEAT`

```sql
DEMILITER $
CREATE PROCEDURE test_repeat(n int)
BEGIN
    declare sum int default 0;
    REPEAT
        SET sum=sum+n;
        SET n=n-1;
    UNTIL n = 0
    END REPEAT;
    SELECT sum AS result;
END$
```

- `LOOP`

```sql
DEMILITER $
CREATE PROCEDURE test_loop(n int)
BEGIN
    declare sum int default 0;
    L: LOOP
        SET sum=sum+n;
        SET n=n-1;
        IF n=0 THEN
            leave L;
        END IF;
    END LOOP L;
    SELECT sum;
END$ 
```
