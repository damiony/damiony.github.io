# 【MySQL】存储过程和函数


## 什么是存储过程

存储过程，是经过编译并存储在数据库中的一段`SQL`语句集合。

存储过程和函数的区别，在于存储过程无返回值，而函数有返回值。


## 存储过程的好处

1. 提高了代码的重用性。

2. 减少了`SQL`语句的编译次数，提高了效率。


## 存储过程的语法

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

- 如果存储过程体只有一句话，`BEGIN`和`END`可以省略。

- 存储过程体中，每句`SQL`语句以分号表示结束。

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

```sql
SHOW CREATE PROCEDURE 过程名;
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
CREATE PROCEDURE test_pro1()
BEGIN
    INSERT INTO admin(username, password) VALUES ('a', '1234'), ('b', '1234');
END $
```

调用：

```sql
CALL test_pro1()$
```

### 2. IN

定义：

```sql
DELIMITER $
CREATE PROCEDURE test_pro2(IN username VARCHAR(20), IN password VARCHAR(20))
BEGIN
    DECLARE result INT DEFAULT 0;
    SELECT
        COUNT(*) INTO result
    FROM
        admin
    WHERE
        admin.username = username AND admin.password = password;
    SELECT IF(result>0,'成功','失败');
END $
```

调用：

```sql
CALL test_pro2('abc', '1234')$
```

### 3. OUT

定义：

```sql
DELIMITER $
CREATE PROCEDURE test_pro3(IN a int, IN b int)
BEGIN
    SET b = a*2;
END$
```

调用：

```sql
CALL test_pro3(10, @res)$
SELECT @res$
```

### 5. INOUT

定义：

```sql
DELIMITER $
CREATE PROCEDURE test_pro4(INOUT a INT)
BEGIN
    SET a=a*2
END$
```

调用：

```sql
SET @m=10$
CALL test_pro4(@m)$
```


## 什么是函数

函数和存储过程类似，都是经过编译并存储在数据库中的一段`SQL`语句集合。但是函数有且仅有一个返回值，而存储过程没有返回值。


## 函数的语法

### 1. 创建函数

```sql
CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型
BEGIN
    函数体
END
```

说明：

1. 参数列表包含两部分：参数名和参数类型。

2. 函数体中必须包含`RETURN`。

3. 如果函数体仅有一句话，可以省略`BEGIN`和`END`。

4. 使用`DELIMITER`语句设置函数结束标记。

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

定义：

```sql
DELIMITER $
CREATE FUNCTION test_func1() RETURNS int
BEGIN
    DECLARE a int DEFAULT 10;
    RETURN a;
END$
```

调用：

```sql
SELECT test_func1()$
```

- 有参有返回

定义：

```sql
DELIMITER $
CREATE FUNCTION test_func2(a int) RETURNS int
BEGIN
    SET a = a*2;
    RETURN a;
END$
```

调用：

```sql
SELECT test_func2(10)$
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

在存储过程中，当游标配合循环语句一起使用时，需要知道循环何时结束，此时就需要使用如下机制：

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

