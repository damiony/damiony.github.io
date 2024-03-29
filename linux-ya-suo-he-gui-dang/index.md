# 【Linux】压缩和归档


`Linux`系统中有多个压缩和归档工具，现在介绍常见的几种。

## gzip

`gzip`用于压缩一个或多个文件，具体用法为：

```shell
gzip [options] [file...]
```

### options

常见的选项有：

- `-c`：将内容输出到标准输出，不会生成新文件，压缩文件也不会被删除。

- `-d`：解压缩。

- `-f`：即使压缩文件已经存在，也强制压缩。

- `-h`：显示帮助信息。

- `-l`：显示压缩文件列表。

- `-r`：递归压缩或者解压缩包含在目录中的文件。

- `-t`：校验压缩文件完整性。

- `-v`：压缩时显示详细信息。

- `-k`：不删除源文件。

- `-1`：`--fast`，速度最快的压缩等级。

- `-2 .. -8`：压缩等级，压缩速度从快到慢，压缩比从小到大。

- `-9`：`--best`，压缩比最高的压缩等级。

### 解压缩

`gzip`生成的压缩文件后缀为`.gz`。可以使用`gzip -d`、`-gunzip`和`zcat`解压缩。

解压缩时，会自动查找`.gz`后缀的文件，所以在不引起歧义的情况下，可以省略文件后缀。

`zcat`命令会将结果输出到标准输出，不会生成新的文件，也不会被删除被解压文件。


## bzip2

`bzip2`同样可以压缩文件。

除了`-r`选项，`gzip`支持的功能`bzip2`也同样支持。

`bzip2`生成的压缩文件以`.bz2`为后缀。

可以使用`bzip2 -d`、`bunzip2`和`bzcat`解压缩，`bzcat`的作用和`zcat`也是一样的。


## gzip 和 bzip2 的使用

已经被压缩过的文件，不应该再次被压缩，即使使用的是不同压缩算法。

因为被压缩文件已经不存在冗余信息，如果进行多次压缩，反而会消耗多余的空间来保存压缩信息。

## zip

`zip`即可以进行压缩，又可以进行归档。

### 命令格式

`zip`的命令格式为：

```shell
zip [options] [zipfile [file...]]
```

### options

`zip`的选项参数有很多，现在仅介绍几个常见的：

- `-d`：删除归档文件内的指定文件。

- `-db`：显示已经处理的字节，和剩余的字节。

- `-dc`：显示已压缩的条目数，和剩余的条目数。

- `-e`：压缩时使用密码。

- `-g`：向已存在的归档文件追加内容。

- `-q`：不打印任何信息。

- `-r`：递归压缩，如果不指定，则只会压缩一个空目录。

- `-v`：压缩时打印信息，或者显示版本号。

### 解压缩

使用`unzip`即可对`zip`归档文件进行解压。

如果使用`unzip -l`，则只会显示`zip`归档文件内的包含的文件。


## tar

tar是使用最为普遍的归档工具。

### 命令格式

`tar`的命令格式为：

```shell
tar function [options] object...
```

### function

`function`参数定义了`tar`命令应该做什么。常见的命令参数如下：

- `-A`：将一个`tar`归档文件，追加到另一个`tar`归档文件中。

- `-c`：创建归档文件。

- `-r`：追加文件到归档文件莫问。

- `-t`：列出归档文件的内容。

- `-x`：从已有的归档文件中提取文件。

### options

常见的options有：

- `-f file`：输出到file，或者从file输入。
- `-j`：将输出重定向给`bzip2`，从而进行压缩。
- `-p`：保留所有文件权限。
- `-v`：处理文件时，显示信息。
- `-z`：将输出重定向到`gzip`，从而进行压缩。

注意：通常情况下，`function`参数需要放在`options`之前。

