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

变量由系统提供，属于服务器层面。

### 2. 语法

- 查看所有的系统变量

```mysql
show global | [session] variables;
```

- 查看满足条件的部分系统变量

```mysql
show global | [session] variables like '%char%';
```

- 查看指定的某个系统变量的值

```mysql
select @@global.|[session.]系统变量名;
```

- 为某个系统变量赋值

```mysql
set global | [session] 系统变量名 = 值; 
```

或者

```mysql
set @@global.|[session.]系统变量名 = 值;
```

### 3. 注意

- 如果是全局级别，需要使用global。

- 如果是会话级别，需要加session或者什么都不加。

- 全局变量不能跨重启，即服务器重启后变量值恢复默认。


## 自定义变量

### 1. 什么是自定义变量

用户自定义的，不是系统提供的。

### 2. 用户变量

- 作用域：

  当前会话（连接）有效。

- 声明并初始化：

  `set @变量名=值;`

  或者

  `set @变量名:=值;`

  又或者

  `select @变量名:=值;`

- 赋值：

  ```mysql
  set @变量名=值;
  set @变量名:=值;
  select @变量名:=值;
  ```

  或者

  ```mysql
  select 字段 into @变量名 from 表;
  ```

- 使用

  ```mysql
  select @变量名;
  ```

### 3. 局部变量

- 作用域：

  仅仅在定义它的`begin ... end`中有效。

- 声明：

  ```mysql
  declare 变量名 类型;
  ```

  或者

  ```mysql
  declare 变量名 类型 default 值;
  ```

- 赋值：

  ```mysql
  set 变量名=值;
  set 变量名:=值;
  select @变量名:=值;
  ```

  或者

  ```mysql
  select 字段 into 变量名 from 表;
  ```

- 使用：

  ```mysql
  select 变量名;
  ```

