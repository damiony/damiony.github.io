# 【MySQL】变量


## 变量分类

变量分为**系统变量**和**自定义变量**。

- 系统变量：

  - 全局变量

  - 会话变量

- 自定义变量：

  - 用户变量

  - 局部变量


## 系统变量

### 1. 什么是系统变量

系统变量是由系统提供的，属于服务器层面。

### 2. 语法

- 查看所有的系统变量

```sql
SHOW GLOBAL | [SESSION] variables;
```

- 查看满足条件的部分系统变量

```sql
SHOW GLOBAL | [SESSION] variables like '%char%';
```

- 查看指定的某个系统变量的值

```sql
SELECT @@GLOBAL.|[SESSION.]系统变量名;
```

- 为某个系统变量赋值

```sql
SET GLOBAL | [SESSION] 系统变量名 = 值; 
```

或者

```sql
SET @@GLOBAL.|[SESSION.]系统变量名 = 值;
```

### 3. 注意

- 如果是全局级别，需要使用global。

- 如果是会话级别，需要加session或者什么都不加。

- 全局变量被修改后，服务器重启变量值会恢复默认。


## 自定义变量

### 1. 什么是自定义变量

自定义变量由用户自定义，不由系统提供。

### 2. 用户变量

- 作用域：

  当前会话（连接）有效。

- 声明并初始化：

  `SET @变量名=值;`

  或者：

  `SET @变量名:=值;`

  或者：

  `SELECT @变量名:=值;`

  或者：

  `SELECT 字段 INTO @变量名 FROM 表;`

- 赋值：

  ```sql
  SET @变量名=值;
  SET @变量名:=值;
  SELECT @变量名:=值;
  SELECT 字段 INTO @变量名 FROM 表;
  ```

- 使用

  ```sql
  SELECT @变量名;
  ```

### 3. 局部变量

- 作用域：

  仅仅在定义它的`BEGIN ... END`语句块中有效。

  `BEGIN...END`主要在函数和存储过程中使用。

- 声明：

  ```sql
  DECLARE 变量名 类型;
  ```

  或者

  ```sql
  DECLARE 变量名 类型 DEFAULT 值;
  ```

- 赋值：

  ```sql
  SET 变量名=值;
  SET 变量名:=值;
  ```

  或者

  ```sql
  SELECT 字段 INTO 变量名 FROM 表;
  ```

- 使用：

  ```sql
  SELECT 变量名;
  ```

