# 【MySQL】存储过程和函数


## 什么是存储过程

存储过程，是经过编译并存储在数据库中的一段`SQL`语句集合。

存储过程和函数的区别，在于存储过程无返回值，而函数有返回值。


## 使用存储过程的好处

1. 提高了代码的重用性。

2. 减少了`SQL`语句的编译次数，提高了效率。


## 存储过程语法

### 1. 创建

```sql
CREATE PROCEDURE 过程名(参数列表)
BEGIN
存储过程体
END
```

说明：

- 参数列表包含三部分：参数模式、参数名、参数类型

- 参数模式有三种：`IN`输入，`OUT`输出，`INOUT`输入和输出。

- 如果存储过程体只有一句话，begin和end可以省略。

- 存储过程体中，每句都以分号表示结束。

- 在存储过程的结尾`END`处，需要使用`DELIMITER`设置的结束标记。

### 2. 调用

```sql
CALL 存储过程名(实参列表);
```

### 3. 删除

```sql
DROP PROCEDURE [IF EXISTS] 过程名;
```

### 4. 查看存储过程

```mysql
SHOW CREATER PROCEDURE 过程名
```

或者：

```sql
SHOW PROCEDURE STATUS;
```

还可以从`mysql.proc`中获取存储过程的信息，但是有的版本不支持该方法。

```sql
SELECT * FROM mysql.proc;
```

### 5. 修改

存储过程无法修改。


## 示例

### 1. 无参

定义：

```sql
DELIMITER $
CREATE PROCEDURE myp1()
BEGIN
    INSERT INTO admin(username, `password`)
    VALUES('john1', '0000'), ('lily', '0000');
END $
```

调用：

```sql
CALL myp1()$
```

### 2. IN

```sql
DELIMITER $
CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))
BEGIN
    SELECT bo.*
    FROM boys bo
    RIGHT JOIN beauty b
    ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName
END $
```

### 3. IN

定义：

```sql
DELIMITER $
CREATE PROCEDURE myp4(IN username VARCHAR(20), IN PASSWORD VARCHAR(20))
BEGIN
    DECLARE result INT DEFAULT 0;
    SELECT COUNT(*) INTO result
    FROM admin
    WHERE admin.username = username
    AND admin.password = PASSWORD;      	
    SELECT IF(result>0,'成功','失败');
END $
```

调用：

```sql
CALL myp4('张飞', '8888')$
```

### 4. OUT

定义：

```sql
DELIMITER $
CREATE PROCEDURE myp5(IN beautyName VARCHAR(20), OUT boyName VARCHAR(20))
BEGIN
    SELECT bo.boyName INTO boyName
    FROM boys bo
    INNER JOIN beauty b
    ON bo.id = b.boyfriend_id
    WHERE b.name=beautyName;
END $
```

调用：

```sql
[SET @bName $]
CALL myp5('小昭', @bName) $
SELECT @bName $
```

### 5. INOUT

定义：

```sql
DELIMITER $
CREATE PROCEDURE myp6(INOUT a INT, INOUT b INT)
BEGIN
    SET a=a*2
    SET b=b*2
END $
```

调用：

```sql
SET @m=10$
SET @n=20$
CALL myp6(@m, @n)$
```


## 什么是函数

函数的定义和存储过程一致，都是经过编译并存储在数据库中的一段`SQL`语句集合。

函数有且仅有一个返回值。


## 函数语法

### 1. 创建函数

```sql
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
    函数体
END
```

说明：

1. 参数列表包含两部分：参数名 参数类型。

2. 函数体中必须包含return。

3. 如果函数体仅有一句话，可以省略begin和end。

4. 使用delimiter语句设置函数结束标记。

### 2. 调用

```sql
SELECT 函数名（参数列表）
```

### 3. 查看函数

```sql
SHOW CREATE FUNCTION 函数名;
```

### 4. 删除函数

```sql
DROP FUNCTION 函数名;
```

### 5. 示例

- 无参有返回

```sql
DELIMITER $
CREATE FUNCTION myf1() RETURNS INT
BEGIN
    DECLARE c INT DEFAULT 0;
    SELECT COUNT(*) INTO c
    FROM employees;
    RETURN c;
END $

SELECT myf1()$
```

- 有参有返回

```sql
DELIMITER $
CREATE FUNCTION myf2(empName VARCHAR(20)) RETURNS DOUBLE
BEGIN
    SET @sal=0;
    SELECT salary INTO @sal
    FROM employees
    WHERE name=empName;
    RETURN @sal
END $

SELECT myf2('tom')$
```


## 游标

游标用于存储查询的结果集，主要应用于存储过程和函数。

### 1. 声明

```sql
DECLARE cursor_name CURSOR FOR 查询语句;
```

### 2. OPEN

```sql
OPEN cursor_name;
```

### 3. FETCH

```sql
FETCH cursor_name INTO 变量...;
```

### 4. CLOSE

```sql
CLOSE cursor_name;
```

### 5. `DECLARE EXIT HANDLER FOR NOT FOUND`

在存储过程中，当游标配合循环语句时，需要知道循环合适结束，这时就需要使用如下机制：

```sql
DECLARE EXIT|COUTINUE HANDLER FOR NOT FOUND 语句;
```

该语句的含义是，当游标无法获取下一条数据时，就执行`FOR NOT FOUND`后面的语句。

### 6. 示例

```sql
DELIMITER $
CREATE PROCEDURE test_cursor()
BEGIN
    DECLARE a int(11);
    DECLARE b int(11);
    DECLARE c varchar(10);
    DECLARE over int default 0;

    DECLARE cursor_t CURSOR FOR select * from t1;
    DECLARE EXIT HANDLER FOR NOT FOUND SET over=1;
    OPEN cursor_t;

    WHILE over=0 DO
        FETCH cursor_t INTO a, b, c;
        SELECT a, b, c;
    END WHILE;
    
    CLOSE cursor_t;
END$
```

