# 【MySQL】DML数据操作


## DML

`DML`: (Data Manipulation Language) 数据操作语言。

主要操作有：

1. 插入数据
2. 修改数据
3. 删除数据   


## 插入数据

### 1. 语法

- 插入单行

  ```sql
  INSERT INTO 表名(字段名1，字段名2，......) VALUES(值1，值2，......);
  ```

- 插入多行

  ```sql
  INSERT INTO 表名(字段名1，字段名2，......) VALUES(值1，值2，......),(值1，值2，......)...;
  ```

### 2. 说明

- 字段名列表和字段值列表一一对应。

- 数值型值不用引号，非数值型值必须使用引号。

- 字段名列表可以省略不写，此时字段列表的顺序和定义时一样。

### 3. 示例

```sql
INSERT INTO stuinfo (stuid, stuname, stugender, email, age, majorid)
VALUES(1, 'TOm', '男', 'tom@gmail.com', 12, 1);
```


## 修改数据

### 1. 语法

- 修改单表：

```sql
UPDATE 表名
SET 列=新值, 列=新值,...
WHERE 筛选条件;
```

- 修改多表，sql92语法：

```sql
UPDATE 表名1 别名，表名2 别名
SET 列=值, ...
WHERE 连接条件
AND 筛选条件;
```

- 修改多表，sql99语法：

```sql
UPDATE 表1 别名
INNER|LEFT|RIGHT JOIN 表2 别名
ON 连接条件
SET 列=值,...
WHERE 筛选条件;
```


## 删除数据

### 1. 语法

- DELETE语句：

```sql
DELETE FROM 表名 WHERE 筛选条件;
```

- TRUNCATE语句:

```sql
TRUNCATE TABLE 表名;
```

### 2. `DELETE`和`TRUNCATE`的比较

- `DELETE`可以添加`WHERE`条件，`TRUNCATE`不能添加`WHERE`条件，一次性清除所有数据。

- `TRUNCATE`的效率较高。

- 对于带自增长列的表，使用`DELETE`删除后，重新插入数据时，记录从断点处开始。使用`TRUNCATE`删除后，重新插入数据时，记录从1开始。

- `DELETE`删除数据时，会返回受影响的行数，`TRUNCATE`删除数据时，不会返回受影响的行数。

- `DELETE`删除数据时，可以支持事务回滚，`TRUNCATE`删除数据时，不支持事务回滚。


## 导入数据

### 1. 命令行导入

```bash
mysql -h ip -P port -u username -p 数据库名 < data.sql
```

`data.sql`是包含表结构的备份文件。

### 2. `source`

在`MySQL`客户端中使用`source`，也可以导入数据：

```sql
source data.sql;
```

`data.sql`是包含表结构的数据文件。

### 3. `LOAD DATA`

在`MySQL`客户端，可以使用`LOAD DATA`语句插入数据：

```sql
LOAD DATA LOCAL INFILE '数据文件' INTO TABLE 表名
FIELDS TERMINATED BY '列分隔符'
LINES TERMINATED BY '行分隔符';
```

`LOCAL`表示从客户端主机读取文件，如果省略该关键字，表示从服务器读取文件。

默认情况下，数据文件列的顺序和表字段的顺序是一一对应的，如果想指定字段的顺序，则语法为：

```sql
LOAD DATA LOCAL INFILE '数据文件' INTO TABLE 表名(字段列表...)
FIELDS TERMINATED BY '列分隔符'
LINES TERMINATED BY '行分隔符';
```

### 4. `mysqlimport`

在命令行使用`mysqlimport`工具，也可以将数据插入数据库。

```bash
mysqlimport -h ip -P port -u username -p \
--local \
--fields-terminated-by='列分隔符' \
--lines-terminated-by='字符分隔符' \
--columns=字段列表 \
库名 文件名
```

注意，表名必须和文件名保持一致。

`mysqlimport`选项的含义与`LOAD DATA`语句类似。


## 导出数据

### 1. `SELECT ... INRO OUTFILE`

```sql
SELECT * FROM 表名 INTO OUTFILE '路径' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n'; 
```

说明：

- `FIELDS TERMINATED BY`：指定生成的字段分隔符。

- `ENCLOSED BY`：将每个字段用指定符号包围。

- `LINES TERMINATED BY`：指定生成的行分隔符。

执行上述语句时，需要注意`MySQL`的`secure_file_priv`变量，该变量有三个值：

- `NULL`：不允许生成文件。

- 具体的路径：生成的文件只能放在该路径下。

- 空值：对文件的生成不做限制。

### 2. `mysqldump`

在命令行下使用`mysqldump`，可以实现数据导出备份。用法为：

```bash
mysqldump [options] 库名 [表名]

# 或者
mysqldump [options] --database/-B 库名...

# 或者
mysqldump [options] --all-databases/-A
```

连接选项：

- `-u`：用户名。

- `-p`：密码。

- `-h`：数据库IP。

- `-P`：连接端口。

输出内容选项：

- `--add-drop-database`：在数据库的创建语句前，添加`DROP DATABASE`语句。

- `--add-drop-table`：在表的创建语句前，添加`DROP TABLE`语句，默认开启。

- `--skip-add-drop-table`：表的创建语句前，不添加`DROP TABLE`语句。

- `-n`：不包含数据库的创建语句。

- `-t`：不包含数据表的创建语句。

- `-d`：不包含数据。


