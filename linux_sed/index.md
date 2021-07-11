# 【Linux】sed命令详解


## sed编辑器

sed可以根据命令来编辑数据流，大致处理流程为：

1. 从输入读取一行数据。
2. 按照命令对数据进行匹配。
3. 按照命令修改匹配到的数据。
4. 将新的数据输出到STDOUT。
5. 读取下一行数据，并重复上述过程。
6. 数据流读取完毕，则程序终止。

sed命令格式：

```shell
sed [options] script file
```

options可以是：

- `-e script`: 将script中的命令，添加到已有的命令列表。
- `-f file`: 将file中指定的命令，添加到已有的命令列表。
- `-n`：不输出每一行的处理结果。

**注意：**

`linux`的sed和`mac`的sed，在语法上存在一定差异，本文的测试环境为`linux bash4.2`。

## options的使用示例

假设有一个`data.txt`文件：

```shell
This is line 1.
This is line 2.
```

### 1. 常规命令

```shell
$ sed 's/line/number/' data.txt
this is number 1.
this is number 2.
```

`s/line/number/`是指，将line替换成number。

### 2. 使用`-e`

```shell
$ sed -e 's/line/number/' -e 's/this/This/' data.txt
This is number 1.
This is number 2.
```

### 3. 使用`-f`

```shell
$ cat script.sed
s/line/number/
s/this/This/

$ sed -f script.sed data.txt
This is number 1.
This is number 2.
```

### 4. 使用`-n`

```shell
$ sed -n 's/This/That/' data.txt # 不会有任何输出 
```

## script的基本语法

### 1. 替换`s`

可以使用`s`命令来替换文本，还可以指定替换标记：

```shell
s/pattern/replacement/[flags]
```

替换标记是可选的，具体有以下几种：

- 数字：表明替换第几处匹配的地方，从1开始。

  ```shell
  $ cat data.txt
  this is line 1.
  this is line 2.
  $ sed 's/is/was/2' data.txt
  this was line 1.
  this was line 2.
  ```

- g：替换所有匹配的文本。

  ```shell
  $ cat data.txt
  this is line 1.
  this is line 2.
  $ sed 's/is/*/g' data.txt
  th* * line 1.
  th* * line 2.
  ```

- p：输出被修改后的行，本质上是输出模式空间的内容。

  ```shell
  $ cat data.txt
  this is line 1.
  this is line 2.

  $ sed 's/line 1/*/p' data.txt
  this is *.
  this is *.
  this is line 2.

  $ sed -n 's/line 1/*/p' data.txt
  this is *.
  ```

- w file：将替换的结果输出到file。

  ```shell
  $ cat data.txt
  this is line 1.
  this is line 2.

  $ sed 's/line 1/*/w result.txt' data.txt
  this is *.
  this is line 2.

  $ cat result.txt
  this is *.
  ```

替换命令的分隔符`/`可以换成其他自定义字符：

```shell
$ cat data.txt
this is line 1.
this is line 2.

$ sed 's,line,number,' data.txt
this is number 1.
this is number 2.
```

### 2. 行寻址

sed默认作用于所有行，如果想指定行号，命令格式为：

```shell
[address]command
```

寻址方式有两种。

- 数字方式。

  指定单个行号：

  ```shell
  $ sed '2s/line/number/' data.txt
  this is line 1.
  this is number 2.
  ```

  指定地址区间：

  ```shell
  $ sed '1,2s/line/number/' data.txt
  this is number 1.
  this is number 2.
  ```

  `$`表示最后一行：

  ```shell
  $ sed '1,$s/line/number/' data.txt
  this is number 1.
  this is number 2.
  ```

- 文本模式。

  ```shell
  $ sed '/line 2/s/line/*/' data.txt
  this is line 1.
  this is * 2.
  ```

  文本模式是对模式空间内的文本进行匹配。
  
  并且，文本模式可以使用正则表达式，正则表达式的具体语法此处不做说明。

### 3. 命令组合

如果想执行多条命令，可以使用分号分隔。

```shell
$ sed '1s/line/number/; 1s/is/*/' data.txt
th* is number 1.
this is line 2.
```

可以将命令写成多行。

```shell
$ sed '
> 1,$s/line/number/
> 1,$s/is/*/
> ' data.txt
th* is number 1.
th* is number 2.
```

也可以使用花括号`{}`将多条命令组合在一起。

```shell
$ sed '1,${s/line/number/ ; s/is/*/}' data.txt
th* is number 1.
th* is number 2.
```

### 4. 删除`d`

可以使用删除命令`d`删除指定行，删除时也可以使用行寻址。

```shell
$ sed '1d' data.txt
this is line 2.
```

使用区间寻址时，第一个匹配处打开删除功能，第二个匹配处关闭删除功能。

### 5. 插入`i`

插入命令`i`会在指定行前增加一个新行。

```shell
sed '[address]i\
new line'
```

如果省略`address`，则是在每行前增加新行。

### 6. 追加`a`

追加命令`a`会在指定行后增加一个新行。

```shell
sed '[address]a\
new line'
```

如果省略address，则是在每行后增加新行。

### 7. 修改`c`

修改命令`c`可以修改整行的数据内容。

```shell
$ sed '2c\
> this is new line 2' data.txt
this is line 1.
this is new line 2
```

如果使用区间寻址，c会对整个区间做替换。

如果省略寻址，则对每行数据做修改。

### 8. 转换`y`

转换命令`y`可以对单个字符做一对一映射转换。

```shell
[address]y/chars1/chars2/
```

`chars1`和`chars2`的长度必须一致，`chars1`中的字符，会被转换为`char2`中对应的字符。

```shell
$ sed 'y/hijk/HIJK/' data.txt
tHIs Is lIne 1.
tHIs Is lIne 2.
```

`y`是一个全局命令，即替换所有匹配结果。

### 9. 打印命令p

打印命令p和替换标记p类似，可以打印匹配到的数据行。

```shell
$ sed -n '1p' data.txt
this is line 1.
```

### 10. 打印行号

命令`=`可以打印被匹配数据在文本中的行号。

```shell
$ sed '/2/=' data.txt
this is line 1.
2
this is line 2.
```

### 11. 列出`l`

列出命令`l`可以打印匹配数据，包括文本字符和不可打印的转义字符。

```shell
$ sed -n '1l' data.txt
this is line 1.$ # $是被打印的换行符
```

### 12. 写入命令

`w`命令可以将匹配数据写入文件。

```shell
[address]w filename
```

### 13. 读取数据

读取命令`r`从文件中读取数据，并将文件的所有数据插入到匹配行后面。

```shell
[address]r filename
```

## sed script进阶语法

假设数据文件`data.txt`为：

```shell
this is line 1.
this is line 2.
this is line 3.
this is line 4.
this is line 5.
```

### 1. 模式空间和保持空间

sed在执行命令时，会保存待检查的文本。
 
保存文本时，需要用到两个缓冲区，模式空间和保持空间。读到的数据保存在模式空间，而保持空间主要起辅助作用。

与缓冲区有关的命令：

- h: 将模式空间复制到保持空间。
- H: 将模式空间追加到保持空间。
- g: 将保持空间复制到模式空间。
- G: 将保持空间追加到模式空间。
- x: 交换模式空间和保持空间的内容。

### 2. 单行数据的next命令

单行`next`命令会将匹配数据的下一行，移到sed的模式空间，移动前会清空模式空间。

单行命令是`n`。

```shell
$ sed -n '/line 4/{n; p}' data.txt
this is line 5.
```

### 3. 多行数据的next命令

多行`next`命令，会以追加的方式，将匹配行的下一行添加到模式空间。并且，sed会将模式空间中的数据当成一行处理。

多行命令是`N`。

```shell
$ sed -n '/line 3/{N; p}' data.txt
this is line 3.
this is line 4.
```

当要匹配的数据在多行中时，这种语法就能很好的发挥作用。

### 4. 多行删除命令D

多行删除命令`D`，会删除模式空间中的第一行。

注意，`d`会删除模式空间中的所有数据。

### 5. 多行打印命令P

多行打印命令`P`，会打印模式空间中的第一行。

注意，`p`会打印模式空间中的所有数据。

### 6. 排除命令

排除命令`!`用于将后续命令作用于非匹配行。

```shell
$ sed -n '1,2!p' data.txt
this is line 3.
this is line 4.
this is line 5.
```

### 7. 分支命令

分支命令b可以用来改变命令的执行流程。

```shell
[address]b [label]
```

address用于寻址，label制定了跳转位置，如果省略label，则跳转到命令脚本的结尾。

```shell
$ sed '3b ; s/this is line/*/' data.txt
* 1.
* 2.
this is line 3.
* 4.
* 5.
```

```shell
sed '
> 3b start
> s/this is line/*/
> :start
> s/this is line/jump/
> ' data.txt
* 1.
* 2.
jump 3.
* 4.
* 5.
```

### 8. 测试命令

测试命令`t`会根据替换命令的结果，来决定是否跳转到某个标签。

如果没有指定标签，则跳转到命令结尾。

```shell
sed '
> s/line 3/*/
> t
> s/this is line/*/
> ' data.txt
* 1.
* 2.
this is *.
* 4.
* 5.
```

### 9. 匹配模式

`&`符号代表替换命令中与模式匹配的文本。

```shell
$ echo "The cat sleeps in his hat." | sed 's/.at/"&"/g'
The "cat" sleeps in his "hat".
```

也可以提取某个被匹配项。

```shell
$ echo "That furry cat is pretty" | sed 's/furry \(.at\)/\1/'
That cat is pretty
```

## sed 示例

### 1. 反序输出文本

```shell
$ cat data.txt
this is line 1.
this is line 2.
this is line 3.
this is line 4.
this is line 5.

# tac命令可以反序输出
$ tac data.txt
this is line 5.
this is line 4.
this is line 3.
this is line 2.
this is line 1.

# sed也可以反序输出
$ sed -n '1!G; h; $p' data.txt
this is line 5.
this is line 4.
this is line 3.
this is line 2.
this is line 1.
```

### 2. 加倍行距

```shell
$ cat data.txt
this is line 1.
this is line 2.
this is line 3.

this is line 4.
this is line 5.
$ sed '/^$/d;$!G' data.txt
this is line 1.

this is line 2.

this is line 3.

this is line 4.

this is line 5.
```

### 3. 给文件的行编号

```shell
$ nl data.txt
     1	this is line 1.
     2	this is line 2.
     3	this is line 3.

     4	this is line 4.
     5	this is line 5.

$ cat -n data.txt
     1	this is line 1.
     2	this is line 2.
     3	this is line 3.
     4
     5	this is line 4.
     6	this is line 5.

$ sed '=' data.txt | sed 'N; s/\n/ /'
1 this is line 1.
2 this is line 2.
3 this is line 3.
4
5 this is line 4.
6 this is line 5.
```

### 4. 打印末尾行

```shell
# 打印最后10行
$ sed ':start; N; 11,$D; b start' data.txt
this is line 5.
this is line 6.
this is line 7.
this is line 8.
this is line 9.
this is line 10.
this is line 11.
this is line 12.
this is line 13.
this is line 14.
```

### 5. 删除文本中的连续空白行

```shell
$ cat data.txt
this is line 1.


this is line 2.
this is line 3.
this is line 4.


this is line 5.
$ sed '/./,/^$/!d' data.txt
this is line 1.

this is line 2.
this is line 3.
this is line 4.

this is line 5.
```

### 6. 删除开头空白行

```shell
$ cat data.txt



this is line 1.
this is line 2.
this is line 3.
this is line 4.
this is line 5.
$ sed '/./,$!d' data.txt
this is line 1.
this is line 2.
this is line 3.
this is line 4.
this is line 5.
```

### 7. 删除结尾空白行

```shell
$ cat data.txt
this is line 1.
this is line 2.
this is line 3.
this is line 4.

this is line 5.



$ sed ':start; /^\n*$/{$d; N; b start}' data.txt

this is line 1.
this is line 2.
this is line 3.
this is line 4.

this is line 5.
```

