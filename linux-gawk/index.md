# 【Linux】gawk详解


## 什么是`gawk`

`gawk`是`Unix`中原始`awk`程序的`GNU`版本，可以将`gawk`看成是一种编程语言，并用它处理数据。

`gawk`的执行流程为，首先从标准输入`stdin`或者文件中读取数据，每次读取一行，并针对该行数据执行脚本命令，然后继续读取下一行并重复前面的操作。

## `gawk`命令格式

`gawk`命令格式为：

```shell
gawk options program file
```

### 1. options

常见的`options`为：

- `-F fs`

  指定每行数据的字段分隔符，如`-F;`表示以`;`为字段分隔符。

  默认的字段分隔符是空白字符（空格或者制表符）。

- `-f file`

  `gawk`程序可以预先写在文件中，`-f`用于从指定文件读取程序。

- `-v var=value`

  为`gawk`程序定义变量。

### 2. program

`gawk`的脚本命令必须放在花括号`{}`中，如：

```shell
gawk '{print "hello world"}'
```

程序命令可以有多条，如果多条命令位于同一行，需要用`;`分开。如：

```shell
gawk '{print "hello"; print "world"}'
```

多条命令也可以位于多行，此时，多条命令之间不需要分隔符。如：

```shell
gawk '{
print "hello"
print "world"
}'
```

在对数据进行处理前，可以执行指定命令，这些命令放在`BEGIN`命令块中。如：

```shell
gawk 'BEGIN{print "Title"}'
```

数据处理结束后，也可以执行指定命令，这些命令放在`END`命令块中。如：

```shell
gawk 'END{print "End of File"}'
```

如果只指定了`BEGIN`命令块或者`END`命令块，缺少数据处理命令，则不会对数据做任何操作。

### 3. file

如果`gawk`命令行中没有指定`file`，则从标准输入读取数据，否则就从指定文件读取数据。

如执行命令`gawk '{print "hello world"}'后，程序会从`stdin`接收数据，并打印`hello world`。此时，可以使用`Ctrl + d`组合键来终止程序，该组合键会发送`EOF`字符。

## 变量

### 1. 数据字段变量

每行的数据字段都会对应一个变量，字段是根据字段分隔符来划分的。

- $0：代表整个文本。
- $1：代表第一个字段。
- $2：代表第二个字段。
- $n：代表第n个字段。

### 2. 内建变量

假设`data.txt`是一个文本文件。

- FIELDWIDTHS

  该命令后面会跟上一串数字，如`"3 2 1"`，用来定义每个数据字段的宽度。如：

  ```shell
  gawk 'BEGIN{FIELDWIDTHS="3 2 1"} {print $1, $2, $3}' data.txt
  ```

  使用了`FIELDWIDTHS`后，字段分隔符会被忽略。

- FS

  指定输入数据的字段分隔符。如：

  ```shell
  gawk 'BEGIN{FS=":"} {print $1, $2, $3}' data.txt
  ```

- RS

  指定输入数据的行分隔符。如：

  ```shell
  # 指定空白行为行分隔符
  gawk 'BEGIN{RS=""} {print $0}' data.txt
  ```

- OFS

  指定输出数据的字段分隔符。如：

  ```shell
  gawk 'BEGIN{OFS=";"} {print $1, $2, $3}' data.txt
  ```

- ORS

  指定输出数据的行分隔符。如：

  ```shell
  gawk 'BEGIN{ORS=","} {print $0}' data.txt
  ```

- ARGC

  存放的是当前命令行参数的个数。如：

  ```shell
  gawk '{print ARGC}' data.txt data.txt # 输出是3，脚本命令和选项不参与计数
  ```

- ARGV

  由命令行参数组成的数据。如：

  ```shell
  gawk '{print ARGC, ARGV[0], ARGV[1]}' data.txt
  
  # 2 gawk data.txt
  ```

- ARGIND

  表示当前文件在`ARGV`中的位置。如：

  ```shell
  gawk '{print ARGIND}' data.txt data.txt # 1 2
  ```

- ENVIRON

  由当前`shell`环境变量组成的关联数组。如：

  ```shell
  # 输出HOME目录
  gawk '{print ENVIRON["HOME"]}' data.txt
  ```

- FILENAME

  当前文件的文件名。如：

  ```shell
  gawk '{print FILENAME}' data.txt # data.txt
  ```

- NF

  当前行的字段总数。如：

  ```shell
  gawk '{print NF, $NF}' data.txt
  ```

- FNR

  当前文件已读取的数据行数，遇到新的文件时，会重新从1开始计数。如：

  ```shell
  gawk '{print FNR}' data.txt data.txt # 1 1
  ```

- NR

  已读取的总数据行数，遇到新文件时，会进行累计，而不是重新计数。如：

  ```shell
  gawk '{print NR}' data.txt data.txt # 1 2
  ```

### 3. 自定义变量

变量名由字母、数字、下划线组成，但是不能以数字开头。

自定义变量的定义方式由多种：

- 在程序脚本中定义变量。如：

  ```shell
  gawk 'BEGIN{test="test line"; print test; test=1; print test}'
  # test line
  # 1
  ```

- 在命令行中定义变量，且变量定义位于程序脚本之后。如：

  ```shell
  gawk '{print $n}' n=1 data.txt
  ```

  这种方式定义的变量，不能在`BEGIN`命令块中使用。

- 在命令行中定义变量，且变量定义位于程序脚本之前。此时需要使用`-v`选项。如：

  ```shell
  gawk -v n=2 'BEGIN{print n}'
  ```

## 数组

`gawk`编程语言使用关联数组来实现数组功能。

关联数组的索引值不再必须是连续的数字，而是可以使用任意文本字符串。

### 1. 数组定义

定义方式为：

```shell
arr[index] = value
```

索引查找方式为：

```shell
arr[index]
```

### 2. 遍历数组

数组遍历的语法为：

```shell
for (idx in array)
{
	statements
}
```

如果`statements`只有一行，则可以省略外层的花括号，如：

```shell
for (idx in array)
	statements
```

数组遍历的结果是没有固定顺序的。

### 3. 删除数组中的变量

删除语法为：

```shell
delete array[index]
```

## 算数运算符

`gawk`支持下列算数运算符：

- `+`：加。
- `-`：减。
- `*`：乘。
- `/`：除。
- `%`：取余。
- `^或者**`：乘方。
- `++x`：`x`加1，然后返回`x`。
- `x++`：先返回`x`，然后`x`加一。
- `--x`：`x`减一，然后返回`x`。
- `x--`：先返回`x`，然后`x`减一。

## 模式

在`gawk`程序脚本中，可以使用不同类型的匹配模式，来过滤数据行。

`BEGIN`和`END`可以看成是两个特殊模式。

### 1. 正则表达式

可以使用正则表达式，来对数据行进行匹配。如：

```shell
gawk '/root/{print $1}' /etc/passwd
```

该命令会匹配包含`root`的数据行。

还可以使用匹配操作符`~`，来对某个特定字段进行匹配，如：

```shell
gawk '$1 ~ /^root/{print $0}' /etc/passwd
```

该命令会匹配第一个字段以`root`开头的数据行。

还可以使用`!~`来排除匹配行。如：

```shell
gawk '$1 !~ /^root/{print $0}' /etc/passwd
```

该命令会过滤第一个字段以`root`开头的数据行。

### 2. 数学比较表达式

可以使用的数学比较表达式有：

- x == y
- x <= y
- x < y
- x >= y
- x > y

比较双方，既可以为数字，也可以是任意字符串。

## 流程控制语句

`gawk`支持多种流程控制语句：

- `if...else...`、
- `while...`、
- `do...while...`、
- `for`循环

### 1. if...else...

语法格式为：

```shell
if (condition1)
{
	statement1
} else if (condition2)
{
	statement2
} else
{
	statement3
}
```

如果`statement`只有一行命令，则可以省略外层花括号。还可以将上述命令写成一行，但必须使用分号分隔每个if语句。

### 2. while...

语法格式为：

```shell
while (condition)
{
	statements
}
```

可以使用`break`或者`continue`跳出循环。

### 3. do...while...

语法格式为：

```shell
do
{
	statements
} while (condition)
```

### 4. for循环

`gawk`支持`c`语言风格的`for`循环。语法格式为：

```shell
for (variable assignment; condition; iteration process)
{
	statements
}
```

## `printf`

`gawk`可以使用`printf`进行格式化打印。

`printf`的命令格式为：

```shell
printf "format string", var1, var2...
```

`format string`是格式化字符，其中包含文本字符和占位符。

占位符格式为：`%[modifier]control-letter`，`modifier`是修饰符，`control-letter`是控制字符。

`control-letter`列表如下：

- c：以`ASCII`形式显示。如：

  ```shell
  gawk 'BEGIN{printf "%c\n", 65}'
  # A
  ```

- d：显示整数值。如：

  ```shell
  gawk 'BEGIN{printf "%d\n", 65}'
  # 65
  ```

- i：显示整数值，和`d`一样。如：

  ```shell
  gawk 'BEGIN{printf "%i\n", 65}'
  ```

- e：以科学记数法形式显示一个数。如：

  ```shell
  gawk 'BEGIN{printf "%.2e\n", 100000}'
  # 1.00e+05
  ```

- f：显示浮点数。如：

  ```shell
  gawk 'BEGIN{printf "%f\n", 3.141}'
  # 3.141000
  ```

- g：用科学记数法或浮点数显示（选择较短的格式）。如：

  ```shell
  gawk 'BEGIN{printf "%g\n", 0.0000001}'
  # 1e-07
  ```

- o：显示为八进制。如：

  ```shell
  gawk 'BEGIN{printf "%o\n", 10}'
  # 12
  ```

- s：打印一个字符串。

  ```shell
  gawk 'BEGIN{printf "%s\n", "hello world"}'
  # hello world
  ```

- x：显示为十六进制。如：

  ```shell
  gawk 'BEGIN{printf "%x\n", 10}'
  # a
  ```

- X：显示为十六进制，但是使用大写A~F。

  ```shell
  gawk 'BEGIN{printf "%X\n", 10}'
  # A
  ```

`modifier`如下：

- `width`：输出字段最小宽度。
- `prec`：指定浮点数的最小宽度和精度。
- `-`：文本设置为左对齐，默认是右对齐。
- `+`：文本设置为右对齐。
- `#`：在八进制数前显示`0`，在十六进制前显示`0x`。

