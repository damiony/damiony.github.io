# 【MySQL】DCL用户管理


## DCL含义

`DCL`：数据控制语言`(Data Control Language)`，用来定义访问权限和安全级别。


## MySQL的用户定义

`MySQL`的用户格式为：`username@'白名单'`。

白名单表示允许登陆MySQL的ip地址。如果省略白名单，则等同于`username@'%'`，即对`ip`地址不做限制。

如下表示4个不同用户：

```sql
root@'localhost' # root可以本地登陆MySQL

root@'10.0.0.1' # root可以通过10.0.0.1登陆MySQL

root@'10.0.0.%' # root可以通过10.0.0.x登陆MySQL

root@'%' # 任何ip都能登陆MySQL
```


## 用户管理

### 1. 查看用户列表

用户信息存放在`mysq【l.user`表中。通常情况下，查看`host`, `user`, `authentication_string`即可。

```sql
SELECT user, host, authentication_string FROM mysql.user;
```


### 2. 创建用户

```sql
CREATE mysql.user username@'ip地址' [IDENTIFIED BY password];
```

如果省略`IDENTIFIED BY password`部分，则默认该用户无密码。

### 3. 修改密码

```sql
ALTER mysql.user username@'ip地址' IDENTIFIED BY 'password';
```

### 4. 删除用户

```sql
DROP mysql.user username@'ip地址';
```

### 5. 忘记密码

如果遇到忘记密码的情况，用如下两个参数重启`MySQL`，可以跳过密码验证。

- `--skip-grant-tables`: 跳过授权表。

- `--skip-networking`: 跳过`TCP/IP`连接，避免远程连接（不安全）。

具体处理过程为：

1. 关闭`MySQL`。

2. 重启`MySQL`。

   ```bash
   mysqld_safe --skip-grant-tables --skip-networking
   ```

   或者：

   ```bash
   service mysqld start --skip-grant-tales --skip-networking
   ```

3. 手工加载授权表。

   ```sql
   flush privileges;
   ```

4. 修改密码。

5. 正常模式重启`MySQL`。

   ```sql
   service mysqld restart
   ```   

### 6. 说明

`MySQL8.0`之前，可以使用`grant`命令同时完成建立用户和授权的操作，但是`8.0`之后不行。


## 权限管理

### 1. 查看可授予的权限

```sql
SHOW privileges;
```

### 2. 授权

`8.0`版本之前，使用如下语法：

```sql
GRANT 权限 ON 对象 TO 用户 IDENTIFIED BY '密码';
```

`8.0`版本之后，需要先建立用户才能授权。

```sql
GRANT 权限 ON 对象 TO 用户;
```

权限`ALL`表示除`GRANT`之外的所有权限，如果想授予`GRANT`权限，`8.0`语法如下，`8.0`之前版本同理。

```sql
GRANT 权限 ON 对象 TO 用户 WITH GRANT OPTION;
```

“对象”是指库和表，如`*.*`表示所有库中的所有表。也可以只授予某几个列的权限，用法是：

```sql
GRANT 权限(列字段...) ...
```

### 3. 回收权限

```sql
REVOKE 权限 ON 对象 FROM 用户;
```

如：

```sql
REVOKE select ON test.t1 FROM test_user@'localhost';
```

### 4. 查看单个用户权限

查看自己：

```sql
SHOW GRANTS;
```

查看他人：

```sql
SHOW GRANTS FOR 用户;
```

### 5. 查看所有用户的权限

查看表相关的权限：

```sql
SELECT * FROM mysql.tables_priv;
```

查看库相关的权限：

```sql
SELECT * FROM mysql.db;
```

查看所有用户的权限：

```sql
SELECT * FROM mysql.user;
```

查看列相关权限：

```sql
SELECT * FROM mysql.columns_priv;
```

其他权限详情可以在`mysql`库中查看。


## 总结

`DCL`主要作用是对用户进行管理，保证数据库的数据安全。用户管理和权限管理属于`DCL`的范畴，上述内容即为相关语法的说明。
